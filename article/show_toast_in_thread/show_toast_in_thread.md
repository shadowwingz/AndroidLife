子线程可以弹 Toast 吗？

作为一个合（cai）格（niao）的安卓开发，我的第一反应当然是不能弹，不是都说子线程不能更新 UI 嘛，那到底能不能弹呢？我们试一下就知道了。

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        Toast.makeText(MainActivity.this, "我是 Toast", Toast.LENGTH_SHORT).show();
    }
}).start();
```

在 Activity 的 onCreate 方法中执行上面的代码，不出意外，报错了，果然，我们的直觉是对的，等等，好像有点不对，让我好好看下报错信息：

```java
java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()
 at android.os.Handler.<init>(Handler.java:200)
 at android.os.Handler.<init>(Handler.java:114)
 at android.widget.Toast$TN$2.<init>(Toast.java:336)
 at android.widget.Toast$TN.<init>(Toast.java:336)
 at android.widget.Toast.<init>(Toast.java:103)
 at android.widget.Toast.makeText(Toast.java:256)
 at com.shadowwingz.androidlifedemo.MainActivity$6.run(MainActivity.java:71)
 at java.lang.Thread.run(Thread.java:761)
```

我记得在子线程更新 UI，报的异常不是这个异常，而是...

```java
android.view.ViewRootImpl$CalledFromWrongThreadException: Only the original thread that created a view hierarchy can touch its views.
 at android.view.ViewRootImpl.checkThread(ViewRootImpl.java:6898)
```

既然不是同一个异常，可能 Toast 在子线程弹出会报错的原因，不是因为不能在子线程更新 UI 导致的，可能是有其他原因。

刚刚我们在子线程弹了一个 Toast，它报的异常是我们没有调用 `Loop.prepare()` 方法，那我们就调用一下 `Loop.prepare()`，Loop 的 prepare 方法会创建一个 Loop 对象和一个对应的 MessageQueue 对象，但是光有一个 Loop 对象是没有用的，我们还要让这个 Loop 运转起来，让 Loop 不停的从 MessageQueue 中取消息。

我们修改下代码

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        Looper.prepare();
        Toast.makeText(MainActivity.this, "我是 Toast", Toast.LENGTH_SHORT).show();
        Looper.loop();
    }
}).start();
```

运行一下，我们发现，Toast 居然弹出来了。

