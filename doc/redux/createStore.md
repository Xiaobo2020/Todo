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

### getState

### subscribe

### dispatch

### replaceReducer

### observable

## All
