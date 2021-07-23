# 子线程可以弹 Toast 吗？

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

这说明，子线程是可以弹 Toast 的，那么为什么子线程可以弹 Toast？要回答这个问题，需要对 Android 的消息机制和 Toast 的显示过程有一定的了解，可以先参考 [消息机制源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md) 和 [Toast 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/toast/toast.md) 有大概的了解。

首先，我先大概解释下，在 Toast 的内部有一个 Handler，这个 Handler 在创建的时候，会自动关联当前线程的 Looper，也就是刚刚子线程中我们创建的 Looper，当 NMS 调用 Toast 的 `showNextToastLocked` 来显示 Toast 时，客户端会使用 Handler 把 mShow 这个任务投递到 Handler 关联的消息队列中，也就是子线程 Looper 对应的消息队列，在 mShow 中完成了 Toast 的显示，这个显示是调用 WindowManager.addView 方法显示的，和 Activity 更新 UI 是不同的，所以不受子线程限制。

分析到这里，我们也可以解释，为什么在子线程中弹 Toast，需要创建 Looper，因为显示 Toast 是一个任务，也就是 mShow，它会被投递到消息队列中，等待被 Loop 从消息队列中取出并执行，如果没有 Looper，就没人把 mShow 这个任务取出来执行了，Toast 也就无法显示了。

接着，我们从源码的角度解释一下：

首先，当我们调用 Toast.makeText() 方法的时候，在 Toast 内部会创建一个内部类 TN 对象：

```java
private static class TN extends I
TransientNotification.Stub {
	final Runnable mShow = new Runnable() {
        @Override
        public void run() {
            handleShow();
        }
    };
    
    final Handler mHandler = new Handler();
    
    @Override
    public void show() {
        if (localLOGV) Log.v(TAG, "SHOW: " + this);
        mHandler.post(mShow);
    }
}
```

TN 是一个 Binder，Toast 的显示就是由 NMS（NotificationManagerService） 回调 TN 的 `show` 方法显示出来的，在 TN 的 `show` 方法中，会把 `mShow` 这个任务投递到 Handler 关联的 MessageQueue 中，而 Handler 关联 MessageQueue 默认关联的是当前线程的 MessageQueue。当我们在主线程弹 Toast 的时候，Toast 中的 Handler 会关联到主线程的 MessageQueue，而主线程默认就有一个 MessageQueue，所以在主线程弹 Toast 不会报错，而在子线程中，默认是没有 MessageQueue 的，需要我们手动去创建一个 Looper，Looper 在创建的时候，会自动创建当前线程的 MessageQueue，子线程有了 MessageQueue，Toast 中的 Handler 才能从 MessageQueue 中取出 `mShow` 这个任务来执行，Toast 才能在子线程中弹出来。

> 总结一下，Toast 的显示需要一个 MessageQueue，主线程因为有一个默认的 MessageQueue，所以不需要我们操心，但是子线程默认是没有 MessageQueue 的，需要我们手动创建一个 Looper，Looper 在创建的时候，会自动创建当前线程的 MessageQueue。

我们再回过头，看最开始的报错信息，也就可以理解了。

```
java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()
```

之所以会报这个错，是因为在 Toast 的内部类 TN 内部的 Handler 创建时，会关联当前线程的 Looper，而我们如果没有关联 Looper，就会报错，这个报错信息是 Handler 源码里：

```java
public Handler(Callback callback, boolean async) {
    ......

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    ......
}
```

#### Looper.loop() 在 Toast 前面调用可以吗？

我一开始认为是可以的，但是试了一下，发现不行，Toast 弹不出来，但是也没有报错信息。

我认为可以，是因为我觉得调用 Looper.loop() 方法之后，Looper 就会不停的从 MessageQueue 中取消息出来执行。这时候，弹 Toast，Toast 就会作为一个任务被投递到 MessageQueue 中，然后被 Looper 取出来执行。

但是结果显示，这是不对的。

我们再好好分析一下，分析什么呢？分析一下 Looper 的 loop 方法。

```java
public static void loop() {
    final Looper me = myLooper();
    ......

    for (;;) {
        Message msg = queue.next(); // might block
        ......
        msg.target.dispatchMessage(msg);
        ......
    }
}
```

在 loop 方法中，是一个死循环 `for (;;)`，因为调用 Looper.loop 方法是在子线程调用的，所以不会造成主线程卡顿，但是 loop 之后的代码是无法执行的，所以我们把 Toast 放在 Looper.loop 方法之后，是无法弹出，原因就是 Toast 根本就没有执行。所以，必须要把 Toast 放在 Looper 的 prepare 和 loop 中间，顺序不能错。

- Toast 放在 Looper 的 prepare 后面，Toast 内部的 Handler 才能关联 Looper 中的 MessageQueue
- Toast 放在 Looper 的 loop 前面，才能在死循环开启之前，将 mShow 任务投递到 MessageQueue 中，等 loop 一调用，死循环开启，Looper 就会从 MessageQueue 中取出 mShow 任务来执行，并且，loop 后面的代码就不会再执行了。除非 Looper 退出。

#### 主线程也是先调用 Looper.loop，为什么主线程可以弹 Toast？

那问题又来了，主线程中创建 Looper，也是先 prepare，然后再 loop 的，那为什么我们在主线程弹 Toast 可以弹出来呢？

因为 Android 系统是基于消息机制的，一个 App 的运行，就是运行在 Looper 的 loop 方法，也就是死循环中，这个死循环还在运行，就表示 App 还在运行，死循环停止，就表示 App 死掉了。

App 运行在死循环中，这个死循环会不停的从主线程的 MessageQueue 中取出消息来执行，我们平常在 Activity 中重写的 onCreate 方法就是对应 `LAUNCH_ACTIVITY` 这个消息，而消息是通过 Handler 来发送的。

所以，一旦 Looper 调用了 loop 方法，就开始了死循环，这个时候如果想把消息投递到 MessageQueue 中，只能通过 Handler 来发送消息。