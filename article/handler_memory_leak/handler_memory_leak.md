#### Hanlder 使用不当导致内存泄漏

在开发中，Handler 如果使用不当，是会导致内存泄漏的，比如下面这种写法：

```java
private Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
    }
};
```

这段代码相信大家肯定不会陌生，我刚学 Android 那会也是这么写的，看起来好像没毛病，但是实际上这段时间是有可能引起内存泄漏的，为什么说有可能呢，首先，先介绍些理论知识，在 Java 中，非静态内部类和匿名内部类都会隐式持有当前类的外部引用，我们刚刚创建 Handler 的方式是匿名内部类，所以 Handler 会持有当前 Activity 的隐式引用，如果 Activity 退出的时候，Handler 没有被释放，那么 Activity 也不会被释放，Activity 不释放，Activity 占用的内存就无法被回收，虚拟机能分配的内存就变少了，这样就产生了内存泄漏。

#### Handler 是怎么造成内存泄漏的

刚刚说了，当 Activity 退出，而 Handler 却没有被释放才会造成内存泄漏，那么 Handler 什么情况下不会被释放？

当我们用 Handler 发送一个延时消息后，等到 Activity 退出的时候，这个消息还没有被执行，这时 Handler 就不会被释放，从而造成 Activity 内存泄漏。

那么，为什么 Handler 发送延时消息，这个消息没有被执行，Activity 退出的时候就会内存泄漏呢？

#### 为什么 Handler 发送的延时消息没有执行，会导致内存泄漏？

这个就涉及到 Android 的消息机制了，当我们调用 Handler 的 `sendEmptyMessageDelayed` 方法发送一个消息时，Handler 内部会把消息投递到 MessageQueue 中，

```java
Handler # sendEmptyMessageDelayed

public final boolean sendEmptyMessageDelayed(int what, long delayMillis) {
	// 获取一条消息
    Message msg = Message.obtain();
    msg.what = what;
    return sendMessageDelayed(msg, delayMillis);
}
```

sendMessageDelayed 方法中最终会调用 enqueueMessage 方法，在这个方法中，Handler 会把自身赋值给 Message 的 target 变量，这样，Message 就持有了 Handler 的引用。

Handler 会和 Message 产生关联：

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
	// Handler 会把自身赋值给 Message 的 target 变量，这样，Message 就持有了 Handler 的引用
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

消息被投递到消息队列中，消息队列是个单链表，我们发送的消息会插入单链表的尾部：

```java
boolean enqueueMessage(Message msg, long when) {
    ......
    synchronized (this) {
		......
        if (p == null || when == 0 || when < p.when) {
            ......
        } else {
			......
            msg.next = p; // invariant: p == prev.next
            // 消息插入单链表尾部，MessageQueue 就持有了消息的引用
            prev.next = msg;
        }
		......
    }
    return true;
}
```

消息插入消息队列，也就是单链表尾部，MessageQueue 就持有了消息的引用，而主线程的 MessageQueue 的生命周期是跟随应用的，应用不退出，MessageQueue 就不会销毁，所以一个引用链就出来了：

- MessageQueue 引用了 Message
- Message 引用了 Handler
- Handler 引用了 Activity

我们发送一个延时消息，在 Message 被执行之前，Message 会一直留在 MessageQueue 中，也就是 MessageQueue 会一直持有 Message 的引用，Message 又持有 Handler 的引用，Handler 又隐式持有 Activity 的引用，所以 Activity 退出的时候，如果 Message 还没执行，就会造成内存泄漏。

#### 怎么用代码检测内存泄漏？

检测内存泄漏，一般是用工具来检测，比如 MAT，或者 Android
 Studio 自带的 `Android Monitor`，但是也可以用代码来检测。我们就用代码来检测下刚刚的 Handler 写法，是不是真的会造成 Activity 内存泄漏。
 
先说下大概思路

1. 开一个线程 LeakThread，在后台专门做泄漏检测
2. 在 Application 中，调用 `registerActivityLifecycleCallbacks` 方法注册一个 Activity 生命周期的回调
3. 在 `registerActivityLifecycleCallbacks` 方法的 `onActivityDestoryed` 回调中，将被回收的 Activity 存储到一个弱引用 WeakReference 中，之所以用弱引用，是因为垃圾回收器一旦发现只具有弱引用的对象，不管当前内存够不够用，直接回收对象的内存。所以，如果 Activity 只被我们的弱引用 WeakReference 引用了，那么 Activity 会被回收掉，如果 Activity 还被 Handler 引用了，那么 Activity 就无法被回收。
4. 在 LeakThread 中，每隔一段时间检测一下 `WeakReference.get()` 是否为空，为空就说明 Activity 已被释放。如果超过一段时间，比如 50s，Activity 还没有被回收，那我们就可以推断 Activity 发生了内存泄漏。
5. 如果检测到某个 Activity 发生了内存泄漏，就提示给开发者，开发者再 dump 出 `.prof `文件分析。

具体代码实现，BaseApplication 实现：

```java
public class BaseApplication extends Application {

    private static WeakReference<Activity> weakReference;

    private LeakThread leakThread = new LeakThread();

    @Override
    public void onCreate() {
        super.onCreate();
        leakThread.start();

        registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {

            }

            @Override
            public void onActivityStarted(Activity activity) {

            }

            @Override
            public void onActivityResumed(Activity activity) {

            }

            @Override
            public void onActivityPaused(Activity activity) {

            }

            @Override
            public void onActivityStopped(Activity activity) {

            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {

            }

            @Override
            public void onActivityDestroyed(Activity activity) {
                weakReference = new WeakReference<>(activity);
            }
        });
    }

    public static void checkActivityRef() {
        if (weakReference != null) {
            System.out.println("Activity 被回收 " + (weakReference.get() == null));
        }
    }
}
```

LeakThread 代码如下：

```java
public class LeakThread extends Thread {

    @Override
    public void run() {
        super.run();
        Timer timer = new Timer();
        // 每隔 1 秒钟，检测一下 Activity 有没有被回收
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                BaseApplication.checkActivityRef();
            }
        }, 0, 1000);
    }
}
```

在 Activity 中，我们发送一个延时消息：

```java
private Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
    }
};

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_memory_leak);
    // 延时 3 分钟的消息
    mHandler.sendEmptyMessageDelayed(0, 3 * 60 * 1000);
}
```

由于 Handler 发送了一个延时 3 分钟执行的消息，那么这个 Activity 退出后，3 分钟之内，应该是不会被回收的，我们实验一下，运行一下程序，我们看一下打印的日志：

```
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
```

在日志中，一直打印 Activity 未被回收。

我们再把 Handler 发送延时消息那句代码屏蔽掉，再看一下：

```
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 false
Activity 被回收 true
Activity 被回收 true
Activity 被回收 true
Activity 被回收 true
```

可以看到，Activity 对象大概 5 秒钟之后，就被回收了。

这样，我们就实现了代码检测 Activity 内存泄漏。