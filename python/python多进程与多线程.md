# python 并发编程

### 多进程

##### 开启多进程方式

- os.fork 开启子进程，windows没有fork方法不能使用
- multiProcess 包下，存在Process类创建单个进程，Pool类创建进程池
- 子进程模块`subprocess` ，很多时候，子进程并不是自身，而是一个外部进程。我们创建了子进程后，还需要控制子进程的输入和输出。

##### 进程间通讯

可以通过Queue、pipes来进行消息通讯



### 多线程

python 标准库提供两个模块：`_thread` 和 `threading`, `_thread`是低级模块，`threading` 是高级模块，对`_thread`进行封装。一般我们使用高级模块进行编程

``` python
import time,threading

def loop():
    print('thread %s is running!' % threading.current_thread().name)
    n = 0
    while n < 5:
        print('thread %s >>> %s' % (threading.current_thread().name,n))
        time.sleep(1)
    print('thread %s ended。' % thread.current_thread().name)
    
print('thread %s is running...' % threading.current_thread().name)
t = threading.Thread(target=loop, name='LoopThread')
t.start()
t.join()
print('thread %s ended.' % threading.current_thread().name)
```





