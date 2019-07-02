## Source Time
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

## Analysis
需要说明的是，`compose`函数的存在其实是服务于中间件的，即当我们使用`applyMiddleware`实现功能增强的背后，其实就是利用了`compose`函数将这多个增强函数进行合理的组合。

首先，`compose`函数利用了es6的rest参数，将多个类型为函数的参数收集到一个叫作`funcs`的数组中，然后进行进一步逻辑判断，最终结果都会是返回一个函数，保证输入输出。

1. 当funcs数组中没有函数，即为一个空数组时，传入什么，传出什么，没有操作。
2. 当funcs数组中有一个函数时，返回这个函数。
3. 有意思的是当有多个函数的时候，funcs会调用reduce方法，将多个函数嵌套。

来用例子说话吧：

```javascript
// 初始数据
let data = 1;
// 加1函数
const add1 = v => v + 1;
// 乘2函数
const multi2 = v => v * 2;

// 1. compose没有参数
const func1 = compose();
func1(data); // -> 1

// 2. compose有一个参数，为add1
const func2 = compose(add1);
func2(data); // -> 2，运算顺序为 1 + 1

// 3. compose有两个参数，顺序依次为add1与multi2
const func3 = compose(add1, multi2);
func3(data); // -> 3，运算顺序为 1 * 2 + 1

// 4. compose有两个参数，顺序依次为multi2与add1
const func4 = compose(multi2, add1);
func4(data); // -> 4，运算顺序为 (1 + 1) * 2
```

这里重点关注一下`func3`与`func4`可以看出，接受参数相同但是顺序不同，也会导致结果的不同，很明显，越是在后面的函数，执行的次序越靠前，从源码中的`a(b(...args))`也可以得到印证。

以上其实就是redux中间件的基本实现原理，关于中间件，我们后续再说。

## Currying
聊完了`compose`这个组合函数，顺便来聊一下`柯里化`这个概念吧，因为这个概念在你编写自己的中间件的时候可能会涉及到，下面是来自于百度百科的解释：

> 在计算机科学中，柯里化（Currying）是把接受多个参数的函数变换成接受一个单一参数(最初函数的第一个参数)的函数，并且返回接受余下的参数且返回结果的新函数的技术。

如果需要举例，那么也很简单，就像这样子：

```javascript
// 现在有个add函数接受两个参数a、b，并返回a + b
const add = (a, b) => a + b;
add(1, 2); // -> 3

// 现在用柯里化的思想来实现一下
const addByCurrying = a => b => a + b;
addByCurrying(1)(2); // -> 3
```

可以看到其实使用柯里化就是将原本接受多个参数的函数变为一次只接受一个参数，并返回一个新的函数用来处理剩余的函数，就像是`add`改写为`addByCurrying`后，实现一个`a+b`其实是利用了两个函数调用完成的，第一次接受1，第二次接受2。

讲完了柯里化的含义，那么为什么要费劲套这么多层去实现一个a+b？那肯定是有他的好处的，想了解具体的就移步这里——[详解JS函数柯里化](https://www.jianshu.com/p/2975c25e4d71)，感觉这篇文章中写的很清楚了。


## All
[index](./index.md)
[compose](./compose.md)
[applyMiddleware](./applyMiddleware.md)
[bindActionCreators](./bindActionCreators.md)