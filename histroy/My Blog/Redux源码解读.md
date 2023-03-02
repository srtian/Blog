
### ![DSC_6780.jpg](https://cdn.nlark.com/yuque/0/2019/jpeg/296173/1570534226258-3dc741f9-df9c-4d04-a445-ae6ad9811c79.jpeg#align=left&display=inline&height=3032&name=DSC_6780.jpg&originHeight=3032&originWidth=3608&size=1871886&status=done&width=3608)

### 前言

作为React全家桶的一份子，MVVM中的VM，Redux为react提供了严谨周密的状态管理。但Redux本身是有点难度的，虽然学习了React也有一段时间了，自我感觉算是入了门，也知道redux的大概流程。但其背后诸如createstore,applymiddleware等API背后到底发生了什么事情，我其实还是不怎么了解的，因此最近花了几天时间阅读了Redux的源码，写下文章纪录一下自己看源码的一些理解。(此文章会随着自己对redux的理解的加深持续更新改进)


### 一、源码结构（redux4.0版本）

Redux是出了名的短小精悍（恩，这个形容很贴切），只有2kb大小，且没有任何依赖。它将所有的脏活累活都交给了中间件去处理，自己保持着很好的纯洁性。再加上redux作者在redux的源码上，也附加了大量的注释，因此redux的源码读起来还是不算难的。

先来看看redux的源码结构，也就是src目录下的代码：

![](https://cdn.nlark.com/yuque/0/2019/png/296173/1570534202075-30b13e27-2dd8-4d3f-a3b1-41ff52c4505e.png#align=left&display=inline&height=366&originHeight=366&originWidth=309&size=0&status=done&width=309)

其中utils是工具函数，主要是作为辅助几个核心API，因此不作讨论。<br />（注：由于篇幅的问题，下面代码很多都删除了官方注释，和较长的warn）


### 二、具体组成

index.js是redux的入口函数具体代码如下：


#### 2.1 index.js

```javascript
import createStore from './createStore'
import combineReducers from './combineReducers'
import bindActionCreators from './bindActionCreators'
import applyMiddleware from './applyMiddleware'
import compose from './compose'
import warning from './utils/warning'
import __DO_NOT_USE__ActionTypes from './utils/actionTypes'

function isCrushed() {}
if (
  process.env.NODE_ENV !== 'production' &&
  typeof isCrushed.name === 'string' &&
  isCrushed.name !== 'isCrushed'
) {
  warning(
  )
}

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```

其中isCrushed函数是用于验证在非生产环境下 Redux 是否被压缩，如果被压缩就会给开发者一个 warn 的提示。

在最后index.js 会暴露 createStore, combineReducers, bindActionCreators, applyMiddleware, compose 这几个redux最主要的API以供大家使用。


#### 2.2 creatStore

createStore函数接受三个参数：

- reducer：是一个函数，返回下一个状态，接受两个参数：当前状态 和 触发的 action；
- preloadedState：初始状态对象，可以很随意指定，比如服务端渲染的初始状态，但是如果使用 combineReducers 来生成 reducer，那必须保持状态对象的 key 和 combineReducers 中的 key 相对应；
- enhancer：是store 的增强器函数，可以指定为中间件，持久化 等，但是这个函数只能用 Redux 提供的 applyMiddleware 函数来进行生成

下面就是creactStore的源码，由于整体源码过长，且 subscribe 和 dispatch 函数也挺长的，所以就将 subscribe 和 dispatch 单独提出来细讲。

```javascript
import $$observable from 'symbol-observable'

import ActionTypes from './utils/actionTypes'
import isPlainObject from './utils/isPlainObject'

export default function createStore(reducer, preloadedState, enhancer) {
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }
  // enhancer应该为一个函数
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    //enhancer 接受 createStore 作为参数，对  createStore 的能力进行增强，并返回增强后的  createStore 。
    //  然后再将  reducer 和  preloadedState 作为参数传给增强后的  createStore ，最终得到生成的 store
    return enhancer(createStore)(reducer, preloadedState)
  }
  // reducer必须是函数
  if (typeof reducer !== 'function') {
    throw new Error('Expected the reducer to be a function.')
  }

 // 初始化参数
  let currentReducer = reducer   // 当前整个reducer
  let currentState = preloadedState   // 当前的state,也就是getState返回的值
  let currentListeners = []  // 当前的订阅store的监听器
  let nextListeners = currentListeners // 下一次的订阅
  let isDispatching = false // 是否处于 dispatch action 状态中, 默认为false

  // 这个函数用于确保currentListeners 和 nextListeners 是不同的引用
  function ensureCanMutateNextListeners() {
    if (nextListeners === currentListeners) {
      nextListeners = currentListeners.slice()
    }
  }

  // 返回state
  function getState() {
    if (isDispatching) {
      throw new Error(
        ......
      )
    }
    return currentState
  }

  // 添加订阅
  function subscribe(listener) {
  ......
    }
  }
// 分发action
  function dispatch(action) {
    ......
  }

  //这个函数主要用于 reducer 的热替换，用的少
  function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }
    // 替换reducer
    currentReducer = nextReducer
    // 重新进行初始化
    dispatch({ type: ActionTypes.REPLACE })
  }

  // 没有研究，暂且放着，它是不直接暴露给开发者的，提供了给其他一些像观察者模式库的交互操作。
  function observable() {
    ......
  }

  // 创建一个store时的默认state
  // 用于填充初始的状态树
  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```


##### subscribe

```javascript
function subscribe(listener) {
    if (typeof listener !== 'function') {
      throw new Error('Expected the listener to be a function.')
    }

    if (isDispatching) {
      throw new Error(
        ......
      )
    }

    let isSubscribed = true
    // 如果 nextListeners 和 currentListeners 是一个引用，重新复制一个新的
    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        throw new Error(
          .......
        )
      }
      
      isSubscribed = false
      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      // 从nextListeners里面删除，会在下次dispatch生效
      nextListeners.splice(index, 1)
    }
  }
```

有时候有些人会觉得 store.subscribe 用的很少,其实不然，是 react-redux 隐式的为我们帮我们完成了这方面的工作。subscribe 函数可以给 store 的状态添加订阅监听，一旦我们调用了 dispatch 来分发 action ，所有的监听函数就会执行。而 nextListeners 就是储存当前监听函数的列表，当调用 subscribe，传入一个函数作为参数时，就会给 nextListeners 列表 push 这个函数。同时调用 subscribe 函数会返回一个 unsubscribe 函数，用来解绑当前传入的函数，同时在 subscribe 函数定义了一个 isSubscribed 标志变量来判断当前的订阅是否已经被解绑，解绑的操作就是从 nextListeners 列表中删除当前的监听函数。


##### dispatch

dispatch是redux中一个非常核心的方法，也是我们在日常开发中最常用的方法之一。dispatch函数是用来触发状态改变的，他接受一个 action 对象作为参数，然后 reducer 就可以根据 action 的属性以及当前 store 的状态，来生成一个新的状态，从而改变 store 的状态；

```javascript
function dispatch(action) {
    // action 必须是一个对象
    if (!isPlainObject(action)) {
      throw new Error(
        ......
      )
    }
    // type必须要有属性，不能是undefined
    if (typeof action.type === 'undefined') {
      throw new Error(
        ......
      )
    }
    // 禁止在reducers中进行dispatch，因为这样做可能导致分发死循环，同时也增加了数据流动的复杂度
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }

    try {
      isDispatching = true
    // 将当前的状态和 action 传给当前的reducer，用于生成最新的 state
      currentState = currentReducer(currentState, action)
    } finally {  
      // 派发完毕
      isDispatching = false
    }
    // 将nextListeners交给listeners
    const listeners = (currentListeners = nextListeners)
    // 在得到新的状态后，依次调用所有的监听器，通知状态的变更
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }
    return action
  }
```


#### 2.3 compose.js

compose 可以接受一组函数参数，从右到左来组合多个函数，然后返回一个组合函数。它的源码并不长，但设计的十分巧妙：

```javascript
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

compose函数的作用其实其源码的注释里讲的很清楚了，比如下面这样：

```javascript
compose(funcA, funcB, funcC)
```

其实它与这样是等价的：

```javascript
compose(funcA(funcB(funcC())))
```

ompose 做的只是让我们在写深度嵌套的函数时，避免了代码的向右偏移。


#### 2.4 applyMiddleware

applyMiddleware也是redux中非常重要的一个函数，设计的也非常巧妙，让人叹为观止。

```javascript
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    // 利用传入的createStore和reducer和创建一个store
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    // 让每个 middleware 带着 middlewareAPI 这个参数分别执行一遍
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)
    return {
      ...store,
      dispatch
    }
  }
}
```

通过上面的代码，我们可以看出 applyMiddleware 是个三级**柯里化**的函数。它将陆续的获得三个参数：第一个是 middlewares 数组，第二个是 Redux 原生的 createStore，最后一个是 reducer，也就是上面的...args；

applyMiddleware 利用 createStore 和 reducer 创建了一个 store，然后 store 的 getState 方法和 dispatch 方法又分别被直接和间接地赋值给 middlewareAPI 变量。

其中这一句我感觉是最核心的：

```
dispatch = compose(...chain)(store.dispatch)
```

我特意将compose与applyMiddleware放在一块，就是为了解释这段代码。因此上面那段核心代码中，本质上就是这样的(假设...chain有三个函数)：

```
dispatch = f1(f2(f3(store.dispatch))))
```


#### 2.5 combineReducers

combineReducers 这个辅助函数的作用就是，将一个由多个不同 reducer 函数作为 value 的 object 合并成一个最终的 reducer 函数，然后我们就可以对这个 reducer 调用 createStore 方法了。这在createStore的源码的注释中也有提到过。

并且合并后的 reducer 可以调用各个子 reducer，并把它们返回的结果合并成一个 state 对象。 由 combineReducers() 返回的 state 对象，会将传入的每个 reducer 返回的 state 按其传递给 combineReducers() 时对应的 key 进行命名。

下面我们来看源码，下面的源码删除了一些的检查判断，只保留最主要的源码：

```javascript
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)
  // 有效的 reducer 列表
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]
  const finalReducerKeys = Object.keys(finalReducers)

// 返回最终生成的 reducer
  return function combination(state = {}, action) {
    let hasChanged = false
    //定义新的nextState
    const nextState = {}
    // 1，遍历reducers对象中的有效key，
    // 2，执行该key对应的value函数，即子reducer函数，并得到对应的state对象
    // 3，将新的子state挂到新的nextState对象上，而key不变
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
     // 遍历一遍看是否发生改变，发生改变了返回新的state，否则返回原先的state
    return hasChanged ? nextState : state
  }
}
```


#### 2.6 bindActionCreators

bindActionCreators可以把一个 value 为不同 action creator 的对象，转成拥有同名 key 的对象。同时使用 dispatch 对每个 action creator 进行包装，以便可以直接调用它们。<br />bindActionCreators函数并不常用（反正我还没有怎么用过），惟一会使用到 bindActionCreators 的场景就是我们需要把 action creator 往下传到一个组件上，却不想让这个组件觉察到 Redux 的存在，并且不希望把 dispatch 或 Redux store 传给它。

```javascript
// 核心代码，并通过apply将this绑定起来
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
} 
// 这个函数只是把actionCreators这个对象里面包含的每一个actionCreator按照原来的key的方式全部都封装了一遍，核心代码还是上面的
export default function bindActionCreators(actionCreators, dispatch) {
  // 如果actionCreators是一个函数，则说明只有一个actionCreator，就直接调用bindActionCreator
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }
  // 如果是actionCreator是对象或者null的话，就会报错
  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
    ... ... 
  }
 // 遍历对象，然后对每个遍历项的 actionCreator 生成函数，将函数按照原来的 key 值放到一个对象中，最后返回这个对象
  const keys = Object.keys(actionCreators)
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```


### 小节

看一遍redux，感觉设计十分巧秒，不愧是大佬的作品。这次看代码只是初看，往后随着自己学习的不断深入，还需多加研究，绝对还能得到更多的体会。
