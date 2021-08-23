每个请求都是重新渲染一个vue实例。

每次渲染之前，vue实例所需的状态(数据)都已经预处理好了，所以将数据进行响应式的处理是多余的，所以默认情况下响应式数据时禁用的。

禁用响应式数据还可以避免将数据转为响应式对象时的性能开销。

由于没有动态更新，vue组件的钩子函数只有`beforeCreate`和`created`。

避免在`beforeCreate`和`created`钩子函数中产生全局副作用的代码，比如`setInterval`，`setTimeout`等。

通过工厂函数来为每次请求创建vue实例，同样的规则也适用于router、store、event bus实例。