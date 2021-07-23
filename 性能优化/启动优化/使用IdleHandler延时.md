<!-- TOC -->

- [对三方库进行延时初始化](#对三方库进行延时初始化)
- [Handler.postDelayed 延时](#handlerpostdelayed-延时)
- [IdleHandler 延时](#idlehandler-延时)
- [简单使用](#简单使用)
- [可以用在哪些场景](#可以用在哪些场景)

<!-- /TOC -->

## 对三方库进行延时初始化

在日常开发中，我们一般都会在 Application 的 onCreate 方法中去做一些初始化操作，比如第三方库的初始化。

但是由于 Application 的 onCreate 方法的执行时间是算在 App 的启动过程中的，那么 Application 的 onCreate 方法执行时间过长，肯定会影响 App 的启动时长。

所以如果要优化 App 的启动时长，我们可以考虑对某些三方库的初始化做下延时操作，这样既能满足我们对 App 启动时长的要求，又不会影响 App 三方库的正常使用。

那么，怎么延时呢？

## Handler.postDelayed 延时

说到延时，我们第一反应就是 Handler。使用 `Handler.postDelayed` 可以很轻松的完成一个延时任务。但是 `Handler.postDelayed` 可以满足我们的要求吗？

我们来看一下 `postDelayed` 方法：

```java
public final boolean postDelayed(@NonNull Runnable r, long delayMillis) {
    return sendMessageDelayed(getPostMessage(r), delayMillis);
}
```

可以看到，`postDelayed` 内部实际上是调用了 `sendMessageDelayed` 方法，另外， `postDelayed` 方法需要 2 个参数，第一个参数是 Runnable，也就是待执行的任务，第二个参数是 `delayMillis`，也就是延时时间。

第一个参数好说，我们想延时执行什么任务，就把它包装成一个 Runnable 就可以了，关键是第二个参数 `delayMillis`，我们想让我们的任务延时多久执行？

有的同学可能会说，这个简单，随便延时个 2、3 秒就行了。

额，一般来说，没毛病。但是认真来说，不严谨。

万一 App 启动一秒后就要调用某个三方库的方法呢，如果我们设置的这个三方库要过 3 秒钟才会初始化，那肯定就不行了。

所以我们不能简单粗暴的设定一个固定的延时时间。

那有什么比较好的方法呢？我们可以用 IdleHandler。

## IdleHandler 延时

IdleHandler 是什么？

它的 MessageQueue 中的一个接口：

```java
public final class MessageQueue {
    public static interface IdleHandler {
        boolean queueIdle();
    }
}
```

我们知道，Android 主线程有一个消息队列，这个消息队列平常会处理一些消息，比如 View 的 mesaure、layout、draw 都是一个个消息，当消息队列的消息比较多的时候，界面就会比较卡顿。而当消息队列的消息暂时处理完了之后，就会进入一个空闲的状态，这个时候 IdleHandler 会收到一个回调。

所以如果我们在 IdleHandler 的回调中去初始化三方库的话，可以在不影响 App 自身的启动速度的前提下，最快的初始化我们的三方库。

```java
public class DelayInitDispatcher {

    private DelayInitDispatcher() {}

    private static class DelayInitDispatcherHolder {
        private static DelayInitDispatcher instance = new DelayInitDispatcher();
    }

    public static DelayInitDispatcher getInstance() {
        return DelayInitDispatcherHolder.instance;
    }

    private Queue<Runnable> mDelayTasks = new LinkedList<>();

    private MessageQueue.IdleHandler mIdleHandler = new MessageQueue.IdleHandler() {
        @Override
        public boolean queueIdle() {
            if (mDelayTasks.size() > 0) {
                Runnable runnable = mDelayTasks.poll();
                runnable.run();
            }
            return !mDelayTasks.isEmpty();
        }
    };

    public DelayInitDispatcher addTask(Runnable runnable) {
        mDelayTasks.add(runnable);
        return this;
    }

    public void start() {
        Looper.myQueue().addIdleHandler(mIdleHandler);
    }
}
```

在 DelayInitDispatcher 中，我们定义了一个 mDelayTasks 用来保存我们的延时任务。

当消息队列空闲的时候，queueIdle 方法就会被调用：

```java
public boolean queueIdle() {
    // 注意这里用的是 if 而不是 while
    if (mDelayTasks.size() > 0) {
        Runnable runnable = mDelayTasks.poll();
        runnable.run();
    }
    return !mDelayTasks.isEmpty();
}
```

在 queueIdle 方法中，我们会检测 mDelayTasks 是否为空，如果不为空，说明还有待执行的延时任务，此时我们会取出延时任务并执行。

queueIdle 方法的逻辑很简单，只是有 2 点要注意的：

第一个是判断 mDelayTasks 是否为空时，我们用的是 if 而不是 while。之所以不用 while，是因为我们在执行延时任务的时候，有可能消息队列又会变的忙碌起来，这个时候应该优先去执行消息队列中的消息，等消息队列空闲时，再来执行我们的延时任务。

所以如果我们用了 while，就会破坏这个优先级，假如我们的延时任务比较多，在执行过程中，如果消息队列又来了新的消息，这个时候并不会转去执行新消息，而是继续执行我们的延时任务。

第二个是 queueIdle 方法的返回值，如果返回 true，表示我们还想继续监听消息队列，一旦消息队列空闲下来，我们就可以收到回调。如果返回 false，表示我们不想继续监听消息队列了，因此我们也不会再收到回调。

所以我们用 mDelayTasks 是否为空来作为 queueIdle 方法的返回值，如果 mDelayTasks 不为空，说明还有待执行的延时任务，此时我们应该继续监听消息队列，如果 mDelayTasks 为空，说明我们待执行的延时任务已经执行完了，因此我们没必要继续监听消息队列了。

## 简单使用

在 Application 的 onCreate（或者其它地方），我们可以这样使用：

```java
DelayInitDispatcher
    .getInstance()
    .addTask { LogUtil.d("模拟初始化三方库 1"); }
    .addTask { LogUtil.d("模拟初始化三方库 2") }
    .start()
```

## 可以用在哪些场景

除了 App 启动时三方库延时初始化之外，我们还可以用它在 Activity 的 onCreate 方法中获取 View 宽高。

为什么使用 IdleHandler 可以获取 View 宽高呢？我们前面已经说过了，View 的 measure、layout、draw 都是一个个消息，而 IdleHandler 是在消息队列空闲的时候才会回调，因此 IdleHandler 被回调的时候，View 已经绘制完成了，所以此时可以拿到 View 的宽高。