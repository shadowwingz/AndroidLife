简单使用：

```java
Toast.makeText(this, "a", Toast.LENGTH_SHORT).show();

// 或者
Toast toast = Toast.makeText(this, "a", Toast.LENGTH_SHORT);
toast.show();
toast.cancel();
```

源码分析：

先看 `makeText` 源码：

```java
Toast # makeText

public static Toast makeText(Context context, CharSequence text, @Duration int duration) {
    Toast result = new Toast(context);

    LayoutInflater inflate = (LayoutInflater)
            context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    // Toast 的布局文件是 transient_notification
    View v = inflate.inflate(com.android.internal.R.layout.transient_notification, null);
    // 默认的 Toast 里只有一个 TextView
    TextView tv = (TextView)v.findViewById(com.android.internal.R.id.message);
    tv.setText(text);
    
    // 从布局文件加载出的 View 会赋值给 mNextView
    result.mNextView = v;
    result.mDuration = duration;

    return result;
}
```

从上面的代码中，我们可以发现，原来 Toast，其实就是一个 TextView，也就是上面的 `mNextView`然后，这个 TextView，会显示一定的时间，就会自己消失，这个时间就是 `mDuration`。

所以，总结一下，makeToast 方法，会把我们想要显示的文本设置到 TextView 里，这个 TextView 会显示一定的时间。

那么 TextView 已经创建好了，什么时候显示出来呢？我们继续看，看 `show` 方法：

```java
Toast # show

public void show() {
    if (mNextView == null) {
        throw new RuntimeException("setView must have been called");
    }

    INotificationManager service = getService();
    String pkg = mContext.getOpPackageName();
    TN tn = mTN;
    tn.mNextView = mNextView;

    try {
        service.enqueueToast(pkg, tn, mDuration);
    } catch (RemoteException e) {
        // Empty
    }
}
```

首先，对 mNextView 进行了判空处理，mNextView 就是我们要显示的 Toast，也就是 TextView，如果它为空，那我们就直接抛异常，这个很好理解。

接着调用了 getService 方法，我们看下这个方法：

```java
private static INotificationManager sService;

static private INotificationManager getService() {
    if (sService != null) {
        return sService;
    }
    sService = INotificationManager.Stub.asInterface(ServiceManager.getService("notification"));
    return sService;
}
```

看到 getService 方法里面的 Stub，还有 asInterface，我们很容易的猜到，这里和 Binder 有关，也就是 NotificationManagerService，后面简称 `NMS`。NMS 负责安卓系统的通知管理，我们使用的 Toast，作用就是通知用户有什么事件发生了。所以 Toast 当然也是 NMS 来管理的。

所以，调用 getService 方法，是为了获取 NMS 的实例。

我们接着看 show 方法，调用了 `tn.mNextView = mNextView`，这句代码把我们之前创建的 nNextView 赋值给了 TN 对象的 mNextView 字段。TN 是 Toast 的一个静态内部类：

```java
private static class TN extends ITransientNotification.Stub {
    View mNextView;

    @Override
    public void show() {
        if (localLOGV) Log.v(TAG, "SHOW: " + this);
        mHandler.post(mShow);
    }

    @Override
    public void hide() {
        if (localLOGV) Log.v(TAG, "HIDE: " + this);
        mHandler.post(mHide);
    }
}
```

TN 继承了 `ITransientNotification.Stub`，所以 TN 是远程服务真正干活的类，这也告诉我们，弹一个 Toast 并没有我们想象中的那么简单，而是一个 IPC 过程。

TN 是服务端，内部封装了 show 方法和 hide 方法。也就是说，显示和隐藏 Toast 并不是我们决定的，而是服务端，也就是 NMS 决定的。

接着，调用了 NMS 的 `enqueueToast` 方法，这是一次跨进程调用，客户端远程调用服务端的方法：

```java
NotificationManagerService # enqueueToast

// 第一个参数是当前应用的包名，第二个参数表示回调，第三个参数表示 Toast 的时长
@Override
public void enqueueToast(String pkg, ITransientNotification callback, int duration)
{
    ......

    synchronized (mToastQueue) {
        int callingPid = Binder.getCallingPid();
        long callingId = Binder.clearCallingIdentity();
        try {
            ToastRecord record;
            int index = indexOfToastLocked(pkg, callback);
            // If it's already in the queue, we update it in place, we don't
            // move it to the end of the queue.
            if (index >= 0) {
                record = mToastQueue.get(index);
                record.update(duration);
            } else {
                // Limit the number of toasts that any given package except the android
                // package can enqueue.  Prevents DOS attacks and deals with leaks.
                // 如果是非系统应用，mToastQueue 中最多能同时
                // 存在 50 个（MAX_PACKAGE_NOTIFICATIONS） ToastRecord
                if (!isSystemToast) {
                    int count = 0;
                    final int N = mToastQueue.size();
                    for (int i=0; i<N; i++) {
                         final ToastRecord r = mToastQueue.get(i);
                         if (r.pkg.equals(pkg)) {
                             count++;
                             if (count >= MAX_PACKAGE_NOTIFICATIONS) {
                                 Slog.e(TAG, "Package has already posted " + count
                                        + " toasts. Not showing more. Package=" + pkg);
                                 return;
                             }
                         }
                    }
                }

                Binder token = new Binder();
                mWindowManagerInternal.addWindowToken(token,
                        WindowManager.LayoutParams.TYPE_TOAST);
                // 将 Toast 请求封装为 ToastRecord 对象并将其添加到 mToastQueue 中
                record = new ToastRecord(callingPid, pkg, callback, duration, token);
                mToastQueue.add(record);
                index = mToastQueue.size() - 1;
                keepProcessAliveIfNeededLocked(callingPid);
            }
            // If it's at index 0, it's the current toast.  It doesn't matter if it's
            // new or just been updated.  Call back and tell it to show itself.
            // If the callback fails, this will remove it from the list, so don't
            // assume that it's valid after this.
            if (index == 0) {
                // 通过 showNextToastLocked 方法来显示当前的 Toast
                showNextToastLocked();
            }
        } finally {
            Binder.restoreCallingIdentity(callingId);
        }
    }
}
```

在 NotificationManagerService 中，有一个 ArrayList，叫 mToastQueue，从命名来看，Toast 应该就是被存储到这个 List 中。

在 `enqueueToast` 方法中，主要是封装 Toast 为 ToastRecord，然后把 ToastRecord 添加到 `mToastQueue` 中。

把 Toast 添加到 `mToastQueue` 中后，接下来就是要显示 Toast 了，显示 Toast 是调用 showNextToastLocked 来实现的，我们看下 `showNextToastLocked` 方法：

```java
NotificationManagerService # showNextToastLocked

void showNextToastLocked() {
    // 从 mToastQueue 中取出第一个 ToastRecord
    ToastRecord record = mToastQueue.get(0);
    while (record != null) {
        if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
        try {
            // 调用 callback 的 show 方法来显示 Toast
            record.callback.show(record.token);
            // 显示 Toast 之后，发送一个延时消息来隐藏 Toast 并将其从 mToastQueue 中移除
            scheduleTimeoutLocked(record);
            return;
        } catch (RemoteException e) {
            Slog.w(TAG, "Object died trying to show notification " + record.callback
                    + " in package " + record.pkg);
            // remove it from the list and let the process die
            int index = mToastQueue.indexOf(record);
            if (index >= 0) {
                mToastQueue.remove(index);
            }
            keepProcessAliveIfNeededLocked(record.pid);
            if (mToastQueue.size() > 0) {
                record = mToastQueue.get(0);
            } else {
                record = null;
            }
        }
    }
}
```

在 showNextToastLocked 方法中，调用了 `record.callback.show(record.token)` 方法来显示 Toast，`record.callback` 是 `ITransientNotification` 类型，也就是上文中的 TN，也就是说，服务器远程调用客户端的 show 方法，show 方法是运行在客户端的 Binder 线程池里。

我们看下 show 方法的具体实现：

```java
Toast.TN # show

@Override
public void show() {
    if (localLOGV) Log.v(TAG, "SHOW: " + this);
    mHandler.post(mShow);
}

Toast.TN # mShow

final Runnable mShow = new Runnable() {
    @Override
    public void run() {
        handleShow();
    }
};

Toast.TN # handleShow

public void handleShow() {
    if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
            + " mNextView=" + mNextView);
    if (mView != mNextView) {
        // remove the old view if necessary
        handleHide();
        mView = mNextView;
        ......
        mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
        ......
        if (mView.getParent() != null) {
            if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
            mWM.removeView(mView);
        }
        ......
        mWM.addView(mView, mParams);
        ......
    }
}
```

刚刚我们说了，服务端远程调用客户端的 show 方法，所以 show 方法是运行在 Binder 线程池里的，而显示 Toast 属于一个更新 UI 操作，当然不能在线程池里完成，所以要用 Handler 来切换线程。所以在 TN 的 show 方法中，调用了 `mHandler.post(mShow);`，把 mShow 这个 Runnable 任务投递到 mHandler 所在线程，也就是主线程，关联的消息队列里，这样 mShow 就会在主线程执行了。在 `mShow` 中，调用了 `handleShow` 方法，在这个方法中，调用了 WindowManager 的 addView 方法，把 Toast 显示了出来。

Toast 显示出来之后，过段时间就要消失，我们再来看下让 Toast 消失的代码，也就是 `scheduleTimeoutLocked` 方法：

```java
NotificationManagerService # scheduleTimeoutLocked

private void scheduleTimeoutLocked(ToastRecord r)
{
    mHandler.removeCallbacksAndMessages(r);
    Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
    long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
    mHandler.sendMessageDelayed(m, delay);
}

private final class WorkerHandler extends Handler
{
    @Override
    public void handleMessage(Message msg)
    {
        switch (msg.what)
        {
            case MESSAGE_TIMEOUT:
                handleTimeout((ToastRecord)msg.obj);
                break;
            ......
        }
    }

}

private void handleTimeout(ToastRecord record)
{
    synchronized (mToastQueue) {
        int index = indexOfToastLocked(record.pkg, record.callback);
        if (index >= 0) {
            cancelToastLocked(index);
        }
    }
}

void cancelToastLocked(int index) {
    // 取出 ToastRecord
    ToastRecord record = mToastQueue.get(index);
    try {
        // 调用 callback 的 hide 方法隐藏 Toast
        record.callback.hide();
    } catch (RemoteException e) {
        Slog.w(TAG, "Object died trying to hide notification " + record.callback
                + " in package " + record.pkg);
        // don't worry about this, we're about to remove it from
        // the list anyway
    }

    // 从 mToastQueue 中移除 ToastRecord
    ToastRecord lastToast = mToastQueue.remove(index);
    mWindowManagerInternal.removeWindowToken(lastToast.token, true);

    keepProcessAliveIfNeededLocked(record.pid);
    if (mToastQueue.size() > 0) {
        // Show the next one. If the callback fails, this will remove
        // it from the list, so don't assume that the list hasn't changed
        // after this point.
        // 如果 mToastQueue 不为空，说明还有 Toast 需要显示，就继续显示下一个 Toast
        showNextToastLocked();
    }
}
```

在 scheduleTimeoutLocked 方法中，首先调用了 Handler 的 sendMessageDelayed 方法发送了一个延时消息，这个延时消息就是 Toast 要显示的时长。

接着，会调用 `record.callback.hide()` 方法，也就是 TN 的 hide 方法，来隐藏 Toast，我们看下 hide 方法：

```java
Toast.TN # hide

@Override
public void hide() {
    if (localLOGV) Log.v(TAG, "HIDE: " + this);
    // 将 mHide 任务切换到主线程执行
    mHandler.post(mHide);
}

Toast.TN # mHide

final Runnable mHide = new Runnable() {
    @Override
    public void run() {
        handleHide();
        // Don't do this in handleHide() because it is also invoked by handleShow()
        mNextView = null;
    }
};

Toast.TN # handleHide

public void handleHide() {
    if (localLOGV) Log.v(TAG, "HANDLE HIDE: " + this + " mView=" + mView);
    if (mView != null) {
        // note: checking parent() just to make sure the view has
        // been added...  i have seen cases where we get here when
        // the view isn't yet added, so let's try not to crash.
        if (mView.getParent() != null) {
            if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
            mWM.removeView(mView);
        }

        mView = null;
    }
}
```

hide 方法和 show 方法的实现类似，也是通过 Handler 来切换线程，然后调用 WindowManager 的 removeView 方法来移除 View，也就是隐藏 Toast。

隐藏完 Toast 之后，还有些工作要收尾，就是 `mToastQueue` 队列，因为 Toast 已经隐藏了，所以它就没用了，没用的话就要从队列中移除掉了。

到这里，一个 Toast 的显示和隐藏我们就分析完了，但是还不够，因为一个 Toast 隐藏了之后，如果还有其它的 Toast 的话，要继续显示。也就是 `showNextToastLocked` 方法：

```java
NMS # showNextToastLocked

void showNextToastLocked() {
    ToastRecord record = mToastQueue.get(0);
    while (record != null) {
        if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
        try {
            // 显示 Toast
            record.callback.show();
            // 隐藏 Toast
            scheduleTimeoutLocked(record);
            return;
        } catch (RemoteException e) {
            Slog.w(TAG, "Object died trying to show notification " + record.callback
                    + " in package " + record.pkg);
            // remove it from the list and let the process die
            int index = mToastQueue.indexOf(record);
            if (index >= 0) {
                // 从队列中移除 Toast
                mToastQueue.remove(index);
            }
            keepProcessAliveLocked(record.pid);
            if (mToastQueue.size() > 0) {
                record = mToastQueue.get(0);
            } else {
                record = null;
            }
        }
    }
}
```

在 `showNextToastLocked` 方法中，会从 mToastQueue 队列中取出第一个 Toast，然后调用 `record.callback.show()` 来显示，再调用 `scheduleTimeoutLocked(record)` 来延迟隐藏 Toast。最后调用 `mToastQueue.remove(index)` 从队列中移除 Toast。

### 总结 ###



为什么 Toast 要先和 NMS（NotificationManagerService）跨进程通信，让 NMS 调用 TN 的 show 方法。为什么不直接让 Toast 调用 show 方法。

猜想：Toast 是一个全局的行为，不依赖于某个应用，也不依赖于某个 Activity，所以 Toast 的显示应该由系统来完成，而且 Toast 的源码也说明了这一点，mToastQueue 中最多能存在 50 个 Toast，只要是非系统应用，大家一起共用这 50 个 Toast，从这个角度看，Toast 也应该是由系统（NMS）来调用。

对于非系统应用，每个应用在这个 mToastQueue 中存储的 Toast 不能超过 50 个，否则 Toast 会被丢弃。

因为 Toast 并不是直接存储到 mToastQueue 中的，而是会被封装到 ToastRecord 中，再存储到 mToastQueue 中，而 ToastRecord 中封装了 Toast 和对应的应用包名。当添加一个 Toast 到 mToastQueue 中时，