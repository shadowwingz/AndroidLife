# Handler 的消息队列是怎么退出的？

消息队列退出有两种方式，一种是调用 Looper 的 `quit`，一种是调用 Looper 的 `quitSafely` 方法。

不管是调用 `quit` 方法，还是调用 `quitSafely`，最终都会调用 MessageQueue 的 quit 方法：

```java
void quit(boolean safe) {
    if (!mQuitAllowed) {
        throw new IllegalStateException("Main thread not allowed to quit.");
    }

    synchronized (this) {
        if (mQuitting) {
            return;
        }
        // 将 mQuitting 置为 true
        mQuitting = true;

		// 安全退出
        if (safe) {
            removeAllFutureMessagesLocked();
        } else { // 非安全退出
            removeAllMessagesLocked();
        }

        // We can assume mPtr != 0 because mQuitting was previously false.
        nativeWake(mPtr);
    }
}
```

首先，MessageQueue 中的 `mQuitting` 变量会被置为 true，这个变量很重要，我们知道，在 Looper 中维持着一个 loop 循环，决定这个循环是否结束，就看 MessageQueue 的 `next` 方法返回的消息是否为 `null`，而在 `next` 方法中，就根据 `mQuitting` 这个变量的值决定是否返回 `null`。

所以，`mQuitting` 这个变量的值就决定了 loop 循环是否结束。那么 loop 循环是一方面，还有未处理完的消息怎么办？

如果是非安全退出，也就是调用 `quit` 方法，最终会调用 `removeAllMessagesLocked` 方法：

```java
private void removeAllMessagesLocked() {
    Message p = mMessages;
    while (p != null) {
        Message n = p.next;
        p.recycleUnchecked();
        p = n;
    }
    mMessages = null;
}
```

我们知道，消息队列里的消息，其实是一个单链表，在 removeAllMessagesLocked 方法中，会遍历这个单链表，对这个单链表中的每个消息都执行 `p.recycleUnchecked` 方法来回收消息。回收消息，其实就是把 Message 对象中的字段都置为 `null`。

安全退出，就比较有讲究了，安全退出调用的是 `removeAllFutureMessagesLocked` 方法：

```java
private void removeAllFutureMessagesLocked() {
    final long now = SystemClock.uptimeMillis();
    Message p = mMessages;
    if (p != null) {
    	// 如果 Message 的 when 字段大于当前时间，
    	// 就直接回收消息
        if (p.when > now) {
            removeAllMessagesLocked();
        } else {
            Message n;
            // 继续判断，取队列中所有大于当前时间的消息
            for (;;) {
                n = p.next;
                if (n == null) {
                    return;
                }
                if (n.when > now) {
                    break;
                }
                p = n;
            }
            p.next = null;
            do {
                p = n;
                n = p.next;
                // 将所有所有大于当前时间的消息的消息回收
                p.recycleUnchecked();
            } while (n != null);
        }
    }
}
```

首先，会判断那些还没执行的 Message 的 when 字段，如果大于当前时间，就回收消息，也就是不让这个消息执行了，如果小于当前时间，那就继续执行消息。

关于 Message 的 `when` 字段，是在 MessageQueue 的 `enqueueMessage` 方法中出现的，我们往回追踪一下代码，可以发现在 Handler 的 `sendMessageAtTime` 方法中，给我们展示了 `when` 字段是怎么被赋值的。

```java
public final boolean sendMessageDelayed(Message msg, long delayMillis)
{
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
```

`SystemClock.uptimeMillis() + delayMillis` 就是 when 字段的值，`SystemClock.uptimeMillis()` 是手机开机的时长，`delayMillis` 是消息延时发送的时间，如果我们发送的不是延时消息，`delayMillis` 就是 0。所以，总结一下，`when` 字段默认就是发送消息时的当前时间。

#### 总结

最后我们总结一下，消息队列退出有两种方式。

- 一种是调用 `quit` 方法，这种方法会直接清空消息队列，同时 Looper 也会停止循环。
- 另一种是调用 `quitSafely` 方法，这种方法会清空所有消息的 when 字段大于当前时间的，对于 when 字段小于当前时间的，消息队列会等到这些消息执行完毕才停止循环。