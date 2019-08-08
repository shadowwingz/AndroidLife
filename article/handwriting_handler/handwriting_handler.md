[Demo 地址](https://github.com/shadowwingz/AndroidLifeDemo/tree/master/app/src/main/java/com/shadowwingz/androidlifedemo/handlerdemo)

Handler 算是面试必问的知识点了，Looper、MessageQueue 相信不少童鞋也能说个一二三，那么我们来挑战点更难的：手写 Handler。

要想手写 Handler，我们也要手写 Looper 和 MessageQueue。那我们就要思考下，Handler、Looper 和 MessageQueue 中都有些什么属性和方法，它们之间又是怎么协作的？

#### Handler 里面有什么？ ####

- MessageQueue 的引用

回忆一下我们使用 Handler，都是调用 Handler 的 sendMessage 方法来发送一个消息，接着这个消息会被投递到消息队列，也就是 MessageQueue 中，所以我们知道，Handler 中肯定有 MessageQueue 的引用。

- 处理消息的方法，用来给 Looper 调用，Looper 从 MessageQueue 中取出消息后，调用 Handler 的处理消息的方法来处理消息。

#### Looper 里面有什么？ ####

- MessageQueue 的引用

因为 Looper 需要不停的从 MessageQueue 中取出消息，所以 Looper 中肯定有 MessageQueue 的引用

- ThreadLocal，用来存储每个线程对应的 Looper

#### MessageQueue 里面有什么？ ####

- 阻塞队列，用来存储消息
- 消息入队的方法，用来给 Handler 调用，投递消息到消息队列
- 取消息的方法，用来给 Looper 调用，从消息队列中取消息

#### Message 里面有什么？ ####

- what，消息内容
- Handler，这个消息由哪个 Handler 来处理

#### 详细代码 ####

- Handler

```java
public class Handler {

    private MessageQueue mQueue;

    public Handler() {
        // 初始化 Looper
        Looper.prepare();
        // 获取当前线程的 Looper
        Looper looper = Looper.myLooper();
        // MessageQueue 和 Looper 关联
        mQueue = looper.sMessageQueue;
    }

    public void handleMessage(Message msg) {

    }

    public void dispatchMessage(Message msg) {
        handleMessage(msg);
    }

    public void sendMessage(Message msg) {
        MessageQueue queue = mQueue;
        if (queue != null) {
            // 将消息投递到消息队列
            queue.enqueueMessage(msg);
        }
    }
}
```

- Message

```java
public class Message {

    // 消息内容
    int what;

    // 负责处理消息的 Handler
    Handler target;
}
```

- Looper

```java
public class Looper {

    static MessageQueue sMessageQueue;

    private static ThreadLocal<Looper> sThreadLocal = new ThreadLocal<>();

    public Looper() {
        sMessageQueue = new MessageQueue();
    }

    public static void prepare() {
        // 每个线程只能创建一个 Looper
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        // 创建当前线程的 Looper 对象，存储到 ThreadLocal 中
        sThreadLocal.set(new Looper());
    }

    public static Looper myLooper() {
        return sThreadLocal.get();
    }

    // 不停地从消息队列中取出消息
    public static void loop() {
        for(;;) {
            // 取出消息
            Message msg = sMessageQueue.next();
            if (msg == null) {
                continue;
            }
            // 交给对应的 Handler 处理
            msg.target.dispatchMessage(msg);
        }
    }
}
```

- MessageQueue

```java
public class MessageQueue {

    // 消息队列，用来存储消息
    private BlockingDeque mBlockingDeque = new LinkedBlockingDeque(50);

    // 将 Handler 发送的消息存储起来
    public void enqueueMessage(Message msg) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        mBlockingDeque.push(msg);
    }

    // 从消息队列中取出消息
    public Message next() {
        Message msg = null;
        try {
            msg = (Message) mBlockingDeque.take();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return msg;
    }
}
```