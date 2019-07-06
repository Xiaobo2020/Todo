## Preface
现在都9012了，还谈redux源码，是不是太晚了？回答是，但又不是，毕竟不写出来只会是更晚。
> 当前版本为`redux@4.0.1`，可能会根据版本不同存在些许差异，不过不影响。

## Directory
```
src
|---- utils
|    |---- actionTypes.js
|    |---- isPlainObject.js
|    |---- warning.js
|---- applyMiddle.js
|---- bindActionCreator.js
|---- combineReducers.js
|---- compose.js
|---- createStore.js
|---- index.js
```

## Analysis
### utils
#### actionTypes.js
`actionTypes.js`保存了`redux`中的一些私有action类型，根据名称也很好理解，这里就不过多赘述了。

```javascript
const randomString = () =>
  Math.random()
    .toString(36)
    .substring(7)
    .split('')
    .join('.')

const ActionTypes = {
  INIT: `@@redux/INIT${randomString()}`,
  REPLACE: `@@redux/REPLACE${randomString()}`,
  PROBE_UNKNOWN_ACTION: () => `@@redux/PROBE_UNKNOWN_ACTION${randomString()}`
}
export default ActionTypes
```

#### isPlainObject.js
`isPlainObject.js`用于判断是否是一个简单对象，这里简单的意思是指是否是单纯的`object`，如数组、函数等就不是，主要依靠的是`Object.getPrototypeOf`获取当前对象的原型，通过当前对象的直接原型和最终原型（即`object`）相比较判断是否是简单对象。

```javascript
export default function isPlainObject(obj) {
  if (typeof obj !== 'object' || obj === null) return false

  let proto = obj
  while (Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto)
  }

  return Object.getPrototypeOf(obj) === proto
}
```


#### warning.js
`warning.js`用于提供一个显示警告信息的通用方法，接受一条错误信息，如果支持`console.error`则会先使用这条命令打印警告信息，再抛出一个Error，不过会在方法中被`catch`掉，没有其他操作。

```javascript
export default function warning(message) {
  if (typeof console !== 'undefined' && typeof console.error === 'function') {
    console.error(message)
  }
  try {
    throw new Error(message)
  } catch (e) {}
}
```

### index.js
`index.js`是redux的入口文件，主要负责判断当前是否是正确的生产环境版本，以及最主要的：向用户提供api，如基本的`createStore`就是在这里导出给用户的。

```javascript
import createStore from './createStore'
import combineReducers from './combineReducers'
import bindActionCreators from './bindActionCreators'
import applyMiddleware from './applyMiddleware'
import compose from './compose'
import warning from './utils/warning'
import __DO_NOT_USE__ActionTypes from './utils/actionTypes'

// ... 此处省略了判断环境以及版本的警告提示

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```

## All
+ [index](./index.md)
+ [compose](./compose.md)
+ [applyMiddleware](./applyMiddleware.md)
+ [bindActionCreators](./bindActionCreators.md)
+ [combineReducers](./combineReducers.md)
+ [createStore](./createStore.md)