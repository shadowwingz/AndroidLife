# 子线程中能否弹 Toast

可以看到，我们在 Toast 的前后分别加了 `Looper.prepare()` 和 `Looper.loop()`，那么，为什么要加这两句代码呢？

我们先不加代码试试，直接在子线程弹一个 Toast，看看有没有什么问题。

```kotlin
Thread(Runnable {
    Toast.makeText(this, "子线程的 Toast", Toast.LENGTH_SHORT).show()
}).start()
```

运行起来，发现 app crash 了：

> java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()

意思是说，不能在还没有调用 `Looper.prepare()` 的线程中创建 Handler。我们再看看源码中的相关代码，报错堆栈在 Handler 的构造方法中：

```java
Handler # Handler

mLooper = Looper.myLooper();
if (mLooper == null) {
    throw new RuntimeException(
        "Can't create handler inside thread " + Thread.currentThread()
                + " that has not called Looper.prepare()");
}
```