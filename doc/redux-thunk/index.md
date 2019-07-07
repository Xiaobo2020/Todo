## Preface
首先需要明确的一点是，`redux-thunk`是一个中间件，需要配合redux提供的`applyMiddleware`一起使用，主要是将常规的对象类型的`action`扩展为可接受函数类型的`action`。它可以让原本只支持同步方式的redux扩展为支持异步的方式，这就是它的强大之处，但是这个背后的实现逻辑却是一点都不复杂。
`
> 当前版本为`redux-thunk@2.3.0`，可能会根据版本不同存在些许差异，不过不影响。

## Source Time
```javascript
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

## Analysis
这里首先定义了一个函数`createThunkMiddleware`，这个函数接受一个额外参数`extraArgument`，而作用就像名字说的一样，返回一个函数，这个函数就是我们需要的中间件`thunk`，如果对于中间件的使用`applyMiddleware`感兴趣，也可以戳[这里](../redux/applyMiddleware.md)查看详情。

接着来讲这个返回的函数，也就是`thunk`中间件。和其他中间件的写法一致，利用柯里化接受一系列的参数。常规来说，当`action`为对象类型的时候，应该调用的是：

```javascript
return next(action); // 这里的next即dispatch这个api
```

但是，如果检测到`action`是一个函数，那么就需要特殊处理一下

```javascript
if (typeof action === 'function') {
  return action(dispatch, getState, extraArgument);
}
```

同时根据这段代码我们也不难分析出，适用于thunk的函数类型的`action`是什么样的，比如这样的：

```javascript
const INCREMENT_COUNTER = 'INCREMENT_COUNTER';

function increment() {
  return {
    type: INCREMENT_COUNTER
  };
}

function incrementAsync() {
  return dispatch => {
    setTimeout(() => {
      // Yay! Can invoke sync or async actions with `dispatch`
      dispatch(increment());
    }, 1000);
  };
}
```

上面这段是截取自`redux-thunk`官方的例子，可以看到不同于一般同步方式的action creator——increment返回的是一个包含了type的对象，`incrementAsync`返回的是一个接受dispatch为入参的匿名函数，在这个函数中可以异步的方式调用dispatch，这也是现在异步请求结合thunk最常用的方式。

总结来说，`redux-thunk`扩展了`redux`能够接受的`action`的类型，不过套用redux中文文档中关于异步流的一句话

> 当 middleware 链中的最后一个 middleware 开始 dispatch action 时，这个 action 必须是一个普通对象。

也就是无论如何处理中间的异步过程，最终dispatch的`action`类型需要回归为对象类型，开始同步的方式处理。
