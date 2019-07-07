## Preface
首先需要明确的一点是，`redux-thunk`是一个中间件，需要配合redux提供的`applyMiddleware`一起使用，主要是将常规的对象类型的`action`扩展为可接受函数类型的`action`。
`
> 当前版本为`redux-thunk@2.3.0`，可能会根据版本不同存在些许差异，不过不影响。
