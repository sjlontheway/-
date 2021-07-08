# Scheduler 源码阅读

### *思考 如何设计一个按照优先级来调度模块*

- 任务量大的时候，我们如何避免大批量的任务执行导致浏览器卡顿？
- 如何将保障任务按照优先级来执行？
- 如何保障低优先级的任务能够得到执行，不被一直中断？
- 任务按照什么样的结构来保存，延期执行的任务该如何保存？
- 任务超时该如何处理？

### *为什么需要有调度*

我们经常听到调度系统的概念，当我们有大批量任务需要处理的时候，我们操作系统处理不过来，那就只能分批次处理，分批处理意味着有些任务先执行，有些任务后执行，那如何处理这个顺序，我们就需要调度系统来决策，谁先执行，谁后执行，React Scheduler 正是在做这样一件事情。

### *阅读源码前置知识*

- 本次分析的`Schduler`版本是 `0.20.1`,源码位置位于 React 仓库 `packages/scheduler` 包，核心逻辑文件在 `packages/scheduler/src/forks/Scheduler.js`下，我们分析的主要对象就是此文件

- React Scheduler 是通过执行时间来编排任务，那么如何编排时间呢，`Scheduler` 是通过投递任务的时间来设定任务 `startTime` ，根据任务的优先级设定超时时间 ` expirationTime `。

  **这两个时间是如何使用的呢？**

  实际上任务有两种情况，一种是需要立即执行的，比如说组件更新，一种是延期执行的，比如Concurrent模式下，Suspense 的使用，这意味着我们需要有两个优先级任务队列来分别保存这两种任务，在 Scheduler 中是`taskQueue` 数组与 `timerQueue` 数组来保存，排序的依据是小顶堆， 分别按照  `expirationTime` 与 `startTime`  大小来进行优先级队列排序的。

  

### **源码解析**

​	调度框架一般分为三个步骤，投递任务、任务调度算法、任务调度执行这三个模块，我们的源码解析也是按照上述三个模块来进行分析:

#### 任务投递

任务投递的入口是: ` unstable_scheduleCallback(priorityLevel, callback, options) `函数，函数参数含义如下：

- `priorityLevel`：优先级，`Scheduler` 将优先级分为5层，分别是`ImmediatePriority，UserBlockingPriority，NormalPriority，LowPriority，IdlePriority`
- callback: 任务执行函数
- options: 执行任务可选参数配置，从源码用到参数来看，只有 delay 一个选项，此选项用来区分延期任务与实时任务

此函数主要功能是，初始化任务，根据任务投递时间设置任务 `startTime`，根据任务优先级设置 `expirationTime` , 根据任务执行时机来将任务加入到`taskqueue` 或者 `timerQueue`，检查是否应该重置调度任务或者执行任务。 

这个函数是整个 `Scheduler` 任务工作的入口也是任务工作的起点。下面我们来看具体源码：

``` javascript


function unstable_scheduleCallback(priorityLevel, callback, options) {
  var currentTime = getCurrentTime();
 // 任务是否延期执行
  var startTime;
  if (typeof options === 'object' && options !== null) {
    var delay = options.delay;
    if (typeof delay === 'number' && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
  } else {
    startTime = currentTime;
  }

  // 优先级实际是设定超时时间,一旦任务超时，任务将立即加入执行队列进行执行
  var timeout;
  switch (priorityLevel) {
    case ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      break;
    case UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      break;
    case IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      break;
    case LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      break;
    case NormalPriority:
    default:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      break;
  }

  var expirationTime = startTime + timeout;
//新建任务，内存id自增器
  var newTask = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: -1,
  };

  // 任务延期执行
  if (startTime > currentTime) {
    // This is a delayed task.
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);
    //...... 省略其他代码
 
  } else { // 直接加入任务队列
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);
    // ...... 省略其余代码
    
  }

  return newTask;
}

```



#### 任务调度算法

从上面代码其实已经包括了调度的关键部分了。每个投递的任务会将投递任务的系统时间  `currentTime` 设定为任务开始执行的时间 (` startTime` )，延时执行的任务会 在 `currentTime` 基础上加上延期执行的时间; 根据带进来优先级参数，根据优先级，`startTme`加上不同优先级分配的时间片，设定 `expirationTime` 即任务过期时间。`taskqueue` 的排序是按照 `expirationTime` 来排序来执行任务；`timeQueue` 按照`startTime` 来排序，当第一个任务到指定执行时间了，通过定时计来将 `timeQueue` 队列中待执行任务调度至 `taskqueue` 队列中执行

总结： 任务调度算法是根据任务的优先级，设定一个过期时间，任务队列按照过期时间来进行优先级排序，随着时间的推移，低优先级的任务将逐渐提升它的优先级，将得到执行，假设任务过期，工作队列将强制执行此任务。

#### 任务调度执行

在讲述其工作之前，我们需要先考虑一个问题，`javascript` 是单线程运行更新UI的，为了保障长时间、大量的 Scheduler 任务不阻塞浏览器正常运转的其余任务，应该使用宏任务来执行调度的任务。除此之外，Scheduler 还引入了时间片概念，当某个任务执行超过一定时间时，主动让出资源，交给其余浏览器资源执行，时间片默认大小 5 ms。

调度的入口是在投递任务函数 `unstable_scheduleCallback` 中，接受到投递任务时，分为两种情况，一种是延时执行任务，一种是立即执行任务。下面根据源码来具体分析实现细节，来分析这两个处理过程：

同步任务启动的入口代码如下：

``` javascript

function unstable_scheduleCallback(priorityLevel, callback, options) {
    //--------------- 省略 无关代码 -------------------
// Schedule a host callback, if needed. If we're already performing work,
    // wait until the next time we yield. 
    if (startTime > currentTime) {
            //--------------- 省略 无关代码 -------------------
    } else { // 投递的是立即执行任务
        假设任务队列处于空闲状态，开启任务循环
        if (!isHostCallbackScheduled && !isPerformingWork) {
          isHostCallbackScheduled = true;
          requestHostCallback(flushWork);
        }
    }
}
```

从上述代码中看得出，投递的是立即执行任务后，假设我们任务队列还未开始调度执行，那么我们调用` requestHostCallback(flushWork) `方法来启动实时任务调度执行

`requestHostCallback ` 函数通过执行` schedulePerformWorkUntilDeadline`  函数，而 `schedulePerformWorkUntilDeadline `通过`MessageChannel` 或者 `setImmediate`，将`performWorkUntilDeadline ` 做为回调函数加入事件循环的宏任务队列，具体代码如下：

``` javascript

let schedulePerformWorkUntilDeadline; // 调度任务函数
if (typeof setImmediate === 'function') {
  // Node.js and old IE.
  schedulePerformWorkUntilDeadline = () => {
    setImmediate(performWorkUntilDeadline);
  };
} else {
  const channel = new MessageChannel();
  const port = channel.port2;
  channel.port1.onmessage = performWorkUntilDeadline;
  schedulePerformWorkUntilDeadline = () => {
    port.postMessage(null);
  };
}

function requestHostCallback(callback) {
  scheduledHostCallback = callback;
  if (!isMessageLoopRunning) {
    isMessageLoopRunning = true;
    schedulePerformWorkUntilDeadline();
  }
}

```

`performWorkUntilDeadline` 方法会在宏任务中分配时间片，开始执行Task，执行任务，如果`taskQueue` 还有任务，那么程序将递归调用`schedulePerformWorkUntilDeadline `继续将 `performWorkUntilDeadline` 回调函数加入宏任务队列，代码如下：

``` javascript
const performWorkUntilDeadline = () => {
  if (scheduledHostCallback !== null) { //存在调度任务
    const currentTime = getCurrentTime();
    deadline = currentTime + yieldInterval; // 过期时间,设置任务最长执行时间片
    const hasTimeRemaining = true;
    let hasMoreWork = true;
    try {
      hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);
    } finally {
      if (hasMoreWork) {
        schedulePerformWorkUntilDeadline();
      } else {
        isMessageLoopRunning = false;
        scheduledHostCallback = null;
      }
    }
  } else {
    isMessageLoopRunning = false; // 不存在调度任务，停止循环
  }
  needsPaint = false;
};
```

任务执行的函数 `scheduledHostCallback` 是在 `requestHostCallback` 赋值 `flushwork` 的，`flushWork`先取消对`timerQueue`的回调，之后设置`isPerformingWork`为`false`，并调用`workLoop`方法执行`taskQueue`中的任务。

``` javascript
function flushWork(hasTimeRemaining, initialTime) {
  isHostCallbackScheduled = false;  // 设置全局标志
  if (isHostTimeoutScheduled) {
    isHostTimeoutScheduled = false;
    cancelHostTimeout();  // 开始执行任务，把timerQueue的超时回调取消
  }

  isPerformingWork = true; // 设置调度任务执行标识
  const previousPriorityLevel = currentPriorityLevel;
  try {
    return workLoop(hasTimeRemaining, initialTime);  // 调用workLoop方法执行taskQueue中任务
  } finally {
    currentTask = null;  // 设置全局标志
    currentPriorityLevel = previousPriorityLevel;  // 设置全局标志
    isPerformingWork = false;  // 设置全局标志
  }
}
```

`workLoop`会先检查 `timeQueue` 延迟队列 中是否有可执行任务，如有将其迁移，然后先不断取出`taskQueue`中的任务，直到**执行完所有任务**或者**执行完所有超时的任务且时间片**已经用完。最后若存在未执行完的任务，则返回`true`。否则重新设置`timerQueue`的超时回调，并返回`false`。

``` javascript
function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  advanceTimers(currentTime); // 任务迁移
  currentTask = peek(taskQueue);
  while (
    currentTask !== null &&
    !(enableSchedulerDebugging && isSchedulerPaused)
  ) {
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost()) // 检测是否让出时间片，注意当该任务执行时间超过截止时间时，任务强制执行
    ) {
      break;
    }
    const callback = currentTask.callback;
    if (typeof callback === 'function') {
      currentTask.callback = null;
      currentPriorityLevel = currentTask.priorityLevel;
      const didUserCallbackTimeout = currentTask.expirationTime <= currentTime;
      if (enableProfiling) {
        markTaskRun(currentTask, currentTime);
      }
      const continuationCallback = callback(didUserCallbackTimeout);
      currentTime = getCurrentTime();
      if (typeof continuationCallback === 'function') {
        currentTask.callback = continuationCallback;
        if (enableProfiling) {
          markTaskYield(currentTask, currentTime);
        }
      } else {
        if (enableProfiling) {
          markTaskCompleted(currentTask, currentTime);
          currentTask.isQueued = false;
        }
        if (currentTask === peek(taskQueue)) {
          pop(taskQueue);
        }
      }
      advanceTimers(currentTime);
    } else {
      pop(taskQueue);
    }
    currentTask = peek(taskQueue);
  }
  // Return whether there's additional work
  if (currentTask !== null) {
    return true;
  } else { // 检查任务是否迁移完成
    const firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }
    return false;
  }
}
```

#### 异步任务

若投递的任务是异步任务，则将任务推入`timerQueue`。如果当前`taskQueue`为空且新任务在`timerQueue`中优先级最高，使用`requestHostTimeout`调度`handleTimeout`方法。

```javascript
newTask.sortIndex = startTime;
push(timerQueue, newTask);
if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
    if (isHostTimeoutScheduled) {
        // 如果timerQueue存在超时回调，取消该回调
        cancelHostTimeout();
    } else {
        isHostTimeoutScheduled = true;
    } 
    // 使用 requestHostTimeout 调度 handleTimeout
    requestHostTimeout(handleTimeout, startTime - currentTime);
}
```

`requestHostTimeout`是对`setTimeout`的包装,将 `handleTimeout` 函数以宏任务运行：

```javascript
  requestHostTimeout = function(callback, ms) {
    taskTimeoutID = setTimeout(() => {
      callback(getCurrentTime()); // 此处执行 handleTimeout
    }, ms);
};
```

`handleTimeout` 先进行任务迁移，检查有没有任务到执行时间。判断当前是否在调度`taskQueue`，若没有在调度，则判断`taskQueue`是否为空，如果不为空，则调度`taskQueue`,否则通过递归执行宏任务 `handleTimeout` 调度`timerQueue`。

```javascript
function handleTimeout(currentTime) {
  isHostTimeoutScheduled = false;
  advanceTimers(currentTime);

  if (!isHostCallbackScheduled) {	// 判断当前是否在调度taskQueue
    if (peek(taskQueue) !== null) {	// 调度taskQueue
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    } else {	// 调度timerQueue
      const firstTimer = peek(timerQueue);
      if (firstTimer !== null) {
        requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
      }
    }
  }
}
```

根据上述代码，我们可以总结出Scheduler的流程图:

![调度流程图](C:\Users\admin\Desktop\react-scheduler-flow.svg)



