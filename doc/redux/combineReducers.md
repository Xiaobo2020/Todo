## Source Time
```javascript
export default function combineReducers(reducers) {
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  // ... 过滤无效key，有效的添加到finalReducers上
  const finalReducerKeys = Object.keys(finalReducers)
  // ... 创建异常stateKey缓存对象
  // ... 错误断言
  return function combination(state = {}, action) {
    // ... 有错误断言处理错误断言
    // ... 非生产环境获取异常state警告信息，并弹出警告
    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      // ... 处理返回state为undefined的错误信息
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```

## Analysis
`combineReducers`接受一个`reducers`对象，并返回一个`combination`统一处理`dispatch`触发的`action`操作。在`combineReducers`中会进行过滤无效reducer、处理reducer返回undefined无效结果等情况，最终得到一个包含了有效子reducer的`finalReducer`对象以及由此衍生得到的`finalReducerKeys`数组。

在新的可以理解为`rootReducer`——`combination`中，会根据`finalReducerKeys`数组遍历`finalReducer`对象，对每一个可能存在的子reducer都运行一次获取到对应的`nextStateForKey`值，最后合并到`nextState`上，而每一次得到的新`nextStateForKey`都会与之前的`previousStateForKey`比较，用于判断`combination`是直接返回原始state还是返回新得到的`nextState`。

## Example
为了方便理解，可以看个简单的例子
```javascript
const theDefaultReducer = (state = 0, action) => state;
const firstNamedReducer = (state = 1, action) => state;
const secondNamedReducer = (state = 2, action) => state;
const rootReducer = combineReducers({
  theDefaultReducer,
  firstNamedReducer,
  secondNamedReducer
});
const store = createStore(rootReducer);
console.log(store.getState());
// -> {theDefaultReducer: 0, firstNamedReducer: 1, secondNameReducer: 2}
```

## All
[index](./index.md)
[compose](./compose.md)
[applyMiddleware](./applyMiddleware.md)
[bindActionCreators](./bindActionCreators.md)
[combineReducers](./combineReducers.md)
[createStore](./createStore.md)