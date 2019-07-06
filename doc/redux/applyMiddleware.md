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

### 1. 创建applyMiddle函数
`applyMiddleware`函数接受一个中间件的散列，并用rest参数收集到`middlewares`这个数组中，最终会返回一个函数fn1。

### 2. fn1的作用描述
函数fn1只有两个作用，第一是接受唯一的参数`createStore`，没错就是redux提供的`createStore`方法；第二返回一个函数fn2。如果看过前一篇文章[compose](./compose.md)的应该可以看出，这里运用了柯里化的思想。

### 3. fn2的作用描述
函数fn2接受一个rest参数（结合`createStore`方法可以知道其实就是`reducer`和`preloadState`），同时在内部详细处理了如何将多个中间件和`redux`的`dispatch`有机结合，以下几点逐行描述。

#### 3.1 创建store
利用最基本的`createStore`创建一个store备用；

```javascript
const store = createStore(...args);
```

#### 3.2 创建dispatch
本地创建一个`dispatch`函数fn3，目前这个dispatch非常简单，就是抛出一个错误。

```javascript
let dispatch = function fun3() {
  throw new Error(
    `Dispatching while constructing your middleware is not allowed. ` +
      `Other middleware would not be applied to this dispatch.`
  );
};
```

#### 3.3 创建中间件需要的入参
创建`middlewareAPI`，可以看到这个对象中只包含了两个属性：`getState`和`dispatch`。getState返回的是上面创建的store的getState方法，以便在中间件中能够获取到state；而dispatch则是一个新的函数fn4，它接受rest参数args，并在内部调用5中提到的本地创建的dispatch（即fn3），注意这里需要传入参数，哪怕定义dispatch的时候并没有定义。

```javascript
const middlewareAPI = {
  getState: store.getState,
  dispatch: function fn4(...args) {
    dispatch(...args);
  },
};
```

#### 3.4 重头戏——组合中间件为一个链
由于这部分嵌套比较多，所以决定引用一个logger中间件来参照：

```javascript
// logger中间件
// 第一层
const logger = store => {
  // 第二层
  return next => {
    // 第三层
    return action => {
      console.group(action.type);
      console.info('dispatching', action);
      let result = next(action);
      console.log('next state', store.getState());
      console.groupEnd(action.type);
      return result;
    };
  };
}

// source code
const chain = middlewares.map(function fn5(middleware) {
  return middleware(middlewareAPI);
});
dispatch = compose(...chain)(store.dispatch);
```

对applyMiddlewares中接受到的rest参数middlewares使用一个map遍历，处理函数fn5的工作是将middlewareAPI注入到中间件中，以logger为例，这里的middlewareAPI就是logger的入参store，同时返回第二层的函数（以next为入参），所以我们知道，`chain`其实就是把所有的中间件都先行注入middlewareAPI后的以next为入参的函数组成的数组，或者可以说，chain中包含了一系列函数，这些函数都被统一注入了middlewareAPI，并且接受next作为入参。

顺带提一下，由middlewareAPI也可以看出，在编写中间件的时候，能拿到的store的api其实就只有两个——getState和dispatch，并且这里的dispatch还是看上去没什么用的本地创建的西贝货dispatch，只能抛出错误。

接着就用到了上一篇提到的compose函数了，这里是接受chain数组展开的散列作为参数，返回一个由多个中间件嵌套的函数，这个函数接受`store.dispatch`作为入参，参考logger就是第二层中的`next`参数，这样我们就得到了一个处理过的、包含了中间件增强功能的dispatch函数（此时已经将那个只会throw Error的dispatch替换掉了）。

如果看过compose就可以知道，入参的顺序决定执行的顺序，所以我们一般将logger中间件尽可能放在applyMiddleware的后面就可以理解了，为了第一步就执行logger。

### 4. 收尾
最终返回一个对象作为用户创建的store，和普通的由createStore创建的store不同的地方在于，这里提供的dispatch是经过了处理的，从这一点上也能很明确看到之前提过的，中间件的作用就是对普通的dispatch进行了增强。

```javascript
return {
  ...store, // 这个store由createStore在本函数中创建
  dispatch,
};
```

## All
[index](./index.md)
[compose](./compose.md)
[applyMiddleware](./applyMiddleware.md)
[bindActionCreators](./bindActionCreators.md)
[combineReducers](./combineReducers.md)
[createStore](./createStore.md)