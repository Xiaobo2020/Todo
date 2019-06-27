## Source Time
```javascript
function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
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
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

为了便于理解，我将源码中的箭头函数全都改为具名函数(以`fn`加上数字标记)，以便于对照分析:

```javascript
function applyMiddleware(...middlewares) {
  return function fn1(createStore) {
    return function fn2(...args) {
      const store = createStore(...args);
      let dispatch = function fun3() {
        throw new Error(
          `Dispatching while constructing your middleware is not allowed. ` +
            `Other middleware would not be applied to this dispatch.`
        );
      };
      const middlewareAPI = {
        getState: store.getState,
        dispatch: function fn4(...args) {
          dispatch(...args);
        },
      };
      const chain = middlewares.map(function fn5(middleware) {
        return middleware(middlewareAPI);
      });
      dispatch = compose(...chain)(store.dispatch);
      return {
        ...store,
        dispatch,
      };
    };
  };
}
```

## Analysis

在详细分析`applyMiddleware`源码之前，我们需要知道的是，中间件`middleware`的功能是对`redux`提供的`dispatch`进行一些扩展，或者说是增强，比如说最常见的`logger`中间件，它的任务就是在`dispatch`的过程中顺带着实现一下打印日志的任务，那么这样看来，其实`applyMiddleware`就像名字一样，其作用就是告诉`redux`怎么处理这些中间件，或者更具体的说，怎么把中间件运用在`redux`提供的`dispatch`上。

好了，下面开始逐条分析源码（这里便于定位，其实是对照改写后的函数）

1. `applyMiddleware`函数接受一个中间件的散列，并用rest参数收集到`middlewares`这个数组中，最终会返回一个函数fn1。
2. 函数fn1只有两个作用，第一是接受唯一的参数`createStore`，没错就是redux提供的`createStore`方法；第二返回一个函数fn2。如果看过前一篇文章[compose](./compose.md)的应该可以看出，这里运用了柯里化的思想。
3. 函数fn2接受一个rest参数（结合`createStore`方法可以知道其实就是`reducer`和`preloadState`），同时在内部详细处理了如何将多个中间件和`redux`的`dispatch`有机结合，以下几点逐行描述。
4. 