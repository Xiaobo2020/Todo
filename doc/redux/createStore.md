## Source Time
```javascript
import $$observable from 'symbol-observable'
import ActionTypes from './utils/actionTypes'
import isPlainObject from './utils/isPlainObject'

export default function createStore(reducer, preloadedState, enhancer) {
  // 第一个参数为reducer，函数类型
  // 第二个参数应为初始state，类型任意
  // 第三个参数为enhancer，函数类型，即applyMiddleware返回的函数
  if (
    (typeof preloadedState === 'function' && typeof enhancer === 'function') ||
    (typeof enhancer === 'function' && typeof arguments[3] === 'function')
  ) {
    // 若preloadedState和enhancer均为函数，
    // 或者参数个数多余三个且最后两个都是函数类型，
    // 猜测使用了多个enhancer，抛出错误
    throw new Error(
      'It looks like you are passing several store enhancers to ' +
        'createStore(). This is not supported. Instead, compose them ' +
        'together to a single function'
    )
  }

  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    // 若preloadedState为函数类型，且enhancer为空，
    // 猜测无初始state，第二个参数直接为enhancer
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      // 若enhancer存在但不是函数类型，则抛出错误，提示enhancer应该是一个函数
      throw new Error('Expected the enhancer to be a function.')
    }

    // 若enhancer存在并且是函数类型，调用enhancer对递归入参createStore
    // 这里可以结合applyMiddleware理解，enhancer即为applyMiddleware的返回结果
    return enhancer(createStore)(reducer, preloadedState)
  }

  if (typeof reducer !== 'function') {
    // 判断reducer类型，非函数类型就报错
    throw new Error('Expected the reducer to be a function.')
  }

  // 当前reducer
  let currentReducer = reducer
  // 当前state
  let currentState = preloadedState
  // 当前事件监听数组
  let currentListeners = []
  // 下一轮事件监听数组，此处与ensureCanMutateNextListeners相关联
  let nextListeners = currentListeners
  // 判断是否正在dispatch的标志
  let isDispatching = false

  function ensureCanMutateNextListeners() {/* ... */}
  function getState() {/* ... */}
  function subscribe(listener) {/* ... */}
  function dispatch(action) {/* ... */}
  function replaceReducer(nextReducer) {/*  */}
  function observable() {/* ... */}

  // When a store is created, an "INIT" action is dispatched so that every
  // reducer returns their initial state. This effectively populates
  // the initial state tree.
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

## Analysis

由于`createStore`内部比较大，所以这里我将一些内部定义的函数拎出单独描述作用，对于其他的部分可参考中文注释内容。最后的英文注释也很好的描述了在创建store之后通过dispatch一次名字叫作`INIT`的action来进行整个store的内部state初始化。总结一下非函数部分内部的功能就是以下几点内容：

1. 判断入参是否符合要求，即只能最多一个enhancer
2. 兼容第二个参数为enhancer而没有初始state的情况
3. 处理有enhancer的创建逻辑
4. 判断reducer的正确性
5. 定义内部功能变量
6. 定义内部功能函数或提供给用户的API
7. 初始化store，初始化state
8. 导出API

讲完了非函数部分内容，接下来一个一个分析一下在`createStore`中定义的函数。

### ensureCanMutateNextListeners
```javascript
function ensureCanMutateNextListeners() {
  if (nextListeners === currentListeners) {
    nextListeners = currentListeners.slice()
  }
}
```
由于`nextListeners`变量定义的时候是根据`currentListeners`获得的数组引用（参考定义部分），所以为了不影响当前事件监听数组，函数`ensureCanMutateNextListeners`会在需要更改`nextListeners`的时候先判断一下是否是当前事件监听数组的引用，若是，则会用`slice`方法获得一个同样元素的不同数组作为新的`nextListeners`。

### getState
```javascript
function getState() {
  // 获取当前store中state，如果正在dispatch则报错
  if (isDispatching) {
    throw new Error(
      'You may not call store.getState() while the reducer is executing. ' +
        'The reducer has already received the state as an argument. ' +
        'Pass it down from the top reducer instead of reading it from the store.'
    )
  }

  return currentState
}
```
函数`getState`主要用于返回当前store中的state对象，需要注意的是如果当前store正在进行`dispatch`操作，那么就不能获取state，而是抛出一个错误提示。

### subscribe
```javascript
function subscribe(listener) {
  if (typeof listener !== 'function') {
    throw new Error('Expected the listener to be a function.')
  }

  if (isDispatching) {
    throw new Error(
      'You may not call store.subscribe() while the reducer is executing. ' +
        'If you would like to be notified after the store has been updated, subscribe from a ' +
        'component and invoke store.getState() in the callback to access the latest state. ' +
        'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
    )
  }

  let isSubscribed = true

  ensureCanMutateNextListeners()
  nextListeners.push(listener)

  return function unsubscribe() {
    if (!isSubscribed) {
      return
    }

    if (isDispatching) {
      throw new Error(
        'You may not unsubscribe from a store listener while the reducer is executing. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      )
    }

    isSubscribed = false

    ensureCanMutateNextListeners()
    const index = nextListeners.indexOf(listener)
    nextListeners.splice(index, 1)
  }
}
```
这里的`subscribe`函数主要用于添加用户事件的监听，会在`dispatch`更新state后进行调用（详情在`dispatch`部分说明），即将监听事件加入到`nextListeners`。需要注意的是，该函数返回的是一个用于解除监听的`unsubscribe`方法，这里利用的是闭包的经典用法，可以参考学习一下。

### dispatch
```javascript
function dispatch(action) {
  if (!isPlainObject(action)) {
    // 判断是否是符合要求的plain object
    throw new Error(
      'Actions must be plain objects. ' +
        'Use custom middleware for async actions.'
    )
  }

  if (typeof action.type === 'undefined') {
    // 判断是否包含所需的type属性
    throw new Error(
      'Actions may not have an undefined "type" property. ' +
        'Have you misspelled a constant?'
    )
  }

  if (isDispatching) {
    // 利用isDispatching判断当前是否正在进行dipatch操作
    throw new Error('Reducers may not dispatch actions.')
  }

  try {
    isDispatching = true
    currentState = currentReducer(currentState, action)
  } finally {
    isDispatching = false
  }

  // 通过nextListeners获得最新的当前事件监听数组，遍历触发监听事件
  const listeners = (currentListeners = nextListeners)
  for (let i = 0; i < listeners.length; i++) {
    const listener = listeners[i]
    listener()
  }

  // 返回入参action
  return action
}
```
`dispatch`函数可以说是用户接触最多的一个api了，功能非常强大，但是它的实现却是非常好理解的。
1. 首先是对接受的入参——`action`进行类型校验，判断是否是合法的plain object；
2. 其次是判断这个`action`是否包含标示更新类型的`type`属性；
3. 利用isDispatching判断是否正在进行`dispatch`操作，如果是则抛出错误；
4. 更新isDispatching标志，并利用将`action`作为入参传入`currentReducer`得到最新的当前state；
5. 通过`nextListeners`获取最新的事件监听数组，同时遍历触发每个监听事件；
6. 返回入参`action`备用。

### replaceReducer
```javascript
function replaceReducer(nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.')
  }

  currentReducer = nextReducer
  dispatch({ type: ActionTypes.REPLACE })
}
```
`replaceReducer`函数接受一个新的reducer函数——`nextReducer`作为入参，在内部替换掉`currentReducer`，同时主动dispatch一次`REPLACE`类型的私有action，用于应用最新的reducer方法更新state。从我的项目经历来说，这个`replaceReducer`方法以及接下来的`observable`方法，使用频率都不是太高，不知道具体使用场景会是什么样子以及其他人的情况如何。

### observable
```javascript
```

## All
