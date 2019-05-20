![](art/1.jpg)

这篇文章，我们来分析 `EventBus.getDefault().register(this)` 的执行流程，也就是 EventBus 是怎么实现注册订阅者的。