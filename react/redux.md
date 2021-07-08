# Redux 学习文档

## redux是什么

redux是一个数据状态管理工具，`Redux is a predictable state container for JavaScript apps.` ,这是redux的官方描述, Redux 是js应用的可预测的状态管理容器,怎么理解这个问题呢，由于应用的操作是可以通过redux 进行管理，所以用户的一系列操作都是可以被监测管理回溯的，这样的设计思想会使得编写代码的逻辑更清晰

## 为什么要用redux

redux 通常而言是和 react组合一起使用的，我们在使用react时经常会碰到一个问题就是，不同组件之间如何共享状态，react是一个单项数据流

## redux的基本概念

- 不变性

- 基本术语
  
  - Actions： action是描述一个动作行为,比如我们通过一个按钮事件触发一个动作,` store.dispatch(action) `, action 会有type来标识这个行为,payload 来传递这个行为所携带的参数，所以 action 可能会是长成这个样子：
  
  ``` ts
    const addTodoAction = {
        type: 'todos/todoAdded',
        payload: {
            title: 'do something',
            content: 'need to do something!'
        }
    }  
  ```

  - Action Creators:

    动作创建器,动作创建器只是一种写法约定, 告诉我们怎么去创建一个描述Actions ,避免我们书写代码时产生不一致action的问题：

    ```typescript
    // action所写的示例就是addTodo这个方法所创建出来的
    const addTodo = (title,content)=>{
        return {
            type: 'todos/todoAdded',
            payload: {
                title,
                content
            }
        }
    }
    ```

  - Reducers

    reducer 是一个以当前state与action为参数的函数,他根据 action 产生的变化来更新state,` (state, action) => newState `;

    reducer 有几个重要的准则必须要遵守:
    - 只能基于action 与 state 参数来计算出新的state
    - 不能修改参数中传递进来的state,产生新的行为后必须创建一个新的state, 必须遵守不可变更新准则(immutable)
    - 不能做任何异步逻辑操作(异步可能会导致 pre action在后一个action之后执行)，计算随机值，或者引起副作用的逻辑
  
    一个reducer的实现逻辑如下代码：

    ``` ts
    const initialState = { value: 0 }
    function counterReducer(state = initialState, action) {
    // Check to see if the reducer cares about this action
    if (action.type === 'counter/increment') {
        // If so, make a copy of `state`
        return {
        ...state,
        // and update the copy with the new value
        value: state.value + 1
        }
    }
    // otherwise return the existing state unchanged
    return state
    }
    ```
  
  - Store
    Redux 应用保存state的地方叫做store，创建方式如下：

    ```ts
    
      import { createStore } from 'redux';
      const store = createStore(reducers)
    
    ```

  - dispatch

    更新数据的唯一方法是 store.dispatch(action),触发一个事件，store调用dispatch后会去触发reducer函数来更新 store的state，我们可以通过store.getState()来使用更新后的state
  
  - Selectors

    selectors 是一个从state取指定数值的函数，当应用变得更复杂时，获取state中的某些字段逻辑会出现多次，使用Selectors函数来封装:

    ``` js

    const selectCounterValue = state => state.value

    const currentValue = selectCounterValue(store.getState())
    console.log(currentValue)

    ```

## redux 数据流

### 早期单向数据流的流程是

- state 描述应用在特定时间点的状态

- UI 根据 state 的值进行渲染

- 用户交互事件改变state

- UI 根据state的改变更新
  
### redux 的数据流

如图所示 ![redux 数据流向图](../images/ReduxDataFlowDiagram.gif)

redux 初始化

- 通过根 reducer 函数创建store对象

- store 初始化调用reducer, 保存初始化state

- UI 第一次渲染时，通过store 获取当前的state 进行渲染, UI 可以订阅的方式来知道 store state 的变化

redux 的更新

- 应用交互事件被触发时，比如说点击按钮

- 应用调用 store 的dispatch action, store.dispatch({type: 'counter/increment'})  

- store 运行 已 当前state与 dispatch action 为参数，执行 reducer 函数，返回一个新的状态
  
- store 通知订阅 store 更新的 UI state 发生改变

- UI 根据 store 更新的state决策是否需要更新
  
- UI 根据 store 更新的state强制更新UI

## redux toolkit使用

### configStore

通过configStore 来创建store, 内部有 combineReducers 函数来将分离的reducer 组合

``` ts
import { configureStore } from '@reduxjs/toolkit'
import counterReducer from '../features/counter/counterSlice'

export default configureStore({
  reducer: {
    counter: counterReducer
  }
})

```

### creating slice Reducer and action

通过 createSliceReducer 将应用store 逻辑按业务分离

``` js
  import { createSlice } from '@reduxjs/toolkit'

export const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0
  },
  reducers: {
    increment: state => {
      state.value += 1
    },
    decrement: state => {
      state.value -= 1
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload
    }
  }
})

export const { increment, decrement, incrementByAmount } = counterSlice.actions
export default counterSlice.reducer

```

#### reducer 不可变更新

在单独使用 redux 时我们有提到 reducer 在更新 state 时, 不能修改 previous state, 必须 新建 state 对象返回，而在 redux Toolkit 中是不需要这样做, 在 createSliceReducer 和 createReducer 方法中已经自动处理，如上例所示,我们直接修改 state 即可,另外reducer函数必须是纯函数
**纯函数: 给定指定参数，返回相同结果**

#### 使用 prepare来预处理payload

在我们处理action时,可能有许多通用的逻辑，这个时候我们可以通过prepare来处理：

``` ts
const postsSlice = createSlice({
  name: 'posts',
  initialState,
  reducers: {
    postAdded: {
      reducer(state, action) {
        state.push(action.payload)
      },
      prepare(title, content) {
        return {
          payload: {
            id: nanoid(),
            title,
            content
          }
        }
      }
    }
    // other reducers here
  }
})

```


### 通过 thunks 来写异步逻辑

我们知道 redux reducer 是不能处理异步逻辑的, 一旦处理异步逻辑，state 更新将不可控

thunks 包含两层函数，内层函数是以 dispatch 与 getState 作为参数，外层是函数创建器，用来 创建以及返回 thunk函数,

redux 的异步逻辑写法通常如下:

``` js
const fetchUserById = userId => {
  // the inside "thunk function"
  return async (dispatch, getState) => {
    try {
      // make an async call in the thunk
      const user = await userAPI.fetchById(userId)
      // dispatch an action when we get the response back
      dispatch(userLoaded(user))
    } catch (err) {
      // If something went wrong, handle it here
    }
  }
}

store.dispatch( fetchUserById(id) );

```

react thunk 中间件会判断传进来的 action 是不是一个 thunk 函数, 如果是thunk 函数则直接将dispatch getState, extraParam 作为参数执行action 函数，否则调用 next(),执行同步逻辑

