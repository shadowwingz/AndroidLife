# IntentService 工作原理

我们知道，当我们启动一个普通的 Service，这个 Service 是运行在主线程的，或者说它的生命周期函数是回调在主线程的，那我们想在 Service 中做一些耗时操作要怎么办呢？有的童鞋可能会说，这个简单，直接在 Service 中 new 一个 Thread 就行了，虽然可以这么做，但是 Android 已经提供给我们一个现成的类，这个类就是 IntentService。

IntentService 继承自 Service，所以 IntentService 的优先级比 Thread 高，内部封装了 Thread，所以比起普通的 Service，适合在后台执行耗时任务，执行完任务后，会自动退出。

```java
public abstract class IntentService extends Service {

    private volatile Looper mServiceLooper;
    // 用于投递任务的 Handler
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;

    // 关联子线程 Looper 的 Handler，
    // 用这个 Handler 发送的消息最终会在子线程被执行
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            // 调用我们重写的 onHandleIntent
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }

    public IntentService(String name) {
        super();
        mName = name;
    }

    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        // 创建 HandlerThread 对象，HandlerThread 继承自 Thread
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        // 启动 Thread，同时在内部会创建 Looper 
        // 并调用其 loop 方法启动消息循环
        thread.start();

        mServiceLooper = thread.getLooper();
        // 让 ServiceHandler 关联子线程的 Looper，
        // 这样，用 ServiceHandler 发送的消息就可以在主线程处理了
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(Intent intent, int startId) {
        // 当我们调用 startService 启动 IntentService 时，
        // 我们传递的 intent 会传到 onStart 方法，接着会赋值给 Message
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        // 发送消息
        mServiceHandler.sendMessage(msg);
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    // 在这个方法中执行耗时操作，耗时操作执行完后，IntentService 会自动退出
    // 这个方法也是我们继承 IntentService 要重写的方法
    protected abstract void onHandleIntent(Intent intent);
}
```

HandlerThread 源码：

```java
public class HandlerThread extends Thread {
    int mPriority;
    int mTid = -1;
    Looper mLooper;

    ......

    @Override
    public void run() {
        mTid = Process.myTid();
        // 创建 Looper
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        // 启动消息队列
        Looper.loop();
        mTid = -1;
    }
    
    ......
 
}
```

首先，讲一下 IntentService 的大概工作流程，也就是当我们调用 startService 启动 IntentServcie，到 onHandleIntent 方法被执行，中间经历了怎样的流程：

当我们继承 IntentService 的时候，需要重写它的 `onHandleIntent` 方法：

```java
public class MyIntentService extends IntentService {
    
    public MyIntentService() {
        super("MyIntentService");
    }

    @Override
    protected void onHandleIntent(@Nullable Intent intent) {

    }
}
```

当我们调用 startService 时：

```java
Intent intent = new Intent(TestActivity.this, MyIntentService.class);
intent.setAction("com.shadowwingz.TEST_INTENT_SERVICE");
startService(intent);
```

这个 intent 会被传递到 IntentService 的 `onStart` 方法中，在 `onStart` 方法里，会把 intent 赋值给 Message，接着把这个 Message 发送到 `mServiceHandler` 关联的消息队列中，最终会被 `mServiceHandler` 的 `handleMessage` 方法处理，在 `handleMessage` 方法中，调用了我们重写的 `onHandleIntent` 方法。

#### 连续启动 3 次 IntentService，任务是按顺序执行吗？为什么？ ####

是按顺序执行，因为我们的任务是被投递到 Handler 关联的消息队列中，Looper 取出来执行的，Looper 取消息是一个一个的取。