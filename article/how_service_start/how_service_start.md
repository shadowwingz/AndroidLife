启动一个 Service 很简单，只需要调用

```java
Intent intent = new Intent(this, TestService.class);
startService(intent);
```

就可以启动一个 Service 了，但是，只会启动 Service 是不够滴，我们还要知道它是怎么启动的，做一个牛（Zhuang）逼的开发者。

和 [Activity 的工作过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_activity_start/how_activity_start.md) 中一样，我们也先提前讲下 Service 启动过程中涉及到的一些对象，以及它们的作用，避免看源码的时候一脸懵逼。

