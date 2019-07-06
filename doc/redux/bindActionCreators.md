## Source Time
```javascript
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}

export default function bindActionCreators(actionCreators, dispatch) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }

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

## Analysis
我们知道，在redux中，`action`是一个plain object，所以为了方便生成这个`action`，我们引入了`action creator`的概念，其实就是一个函数，接受一个参数，返回一个对象，比如下面的`addTodo`：

```javascript
const addTodo = (text) => {
  return {
    text,
    finished: false,
  };
};
```

每当我们想添加一个todo的时候只需要调用`addTodo`并传入名字就行了，但是redux的运行不仅仅止步于此，更关键的是将产生的这个`action`给触发，也就是`dispatch`，所以，`bindActionCreators`就是更进一步，不仅仅是产生`action`，还实现自动`dispatch`。

核心部分是最开始的五行——函数`bindActionCreator`：

```javascript
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}
```

创建一个函数，名字叫作`bindActionCreator`，接受两个参数，一个是`actionCreator`，也就是提到的`addTodo`，第二个参数是`dispatch`，这里的就是store提供的api。而`bindActionCreators`其实是将接受范围扩大化，不仅仅允许actionCreators是函数，也可以是一个对象，这里不过多赘述。

实际使用往往像是这样的:

```javascript
const addTodoBindDispatch = bindActionCreators(addTodo, store.dispatch);
```

> PS. 多说一句，感觉这个api并没有什么卵用

## All
[index](./index.md)
[compose](./compose.md)
[applyMiddleware](./applyMiddleware.md)
[bindActionCreators](./bindActionCreators.md)
[combineReducers](./combineReducers.md)
[createStore](./createStore.md)