Toast 源码解析

简单使用：

```java
Toast.makeText(this, "a", Toast.LENGTH_SHORT).show();

// 或者
Toast toast = Toast.makeText(this, "a", Toast.LENGTH_SHORT);
toast.show();
toast.cancel();
```

源码分析：

先看 makeText 源码：

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

再看 show 方法：

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

可以看到，调用了 NMS（NotificationManagerService）的 enqueueToast 方法，

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

总结一下，enqueueToast 的主要工作就是 封装 Toast 为 ToastRecord，然后把 ToastRecord 添加到 mToastQueue 中，最后调用 showNextToastLocked 来显示 Toast。

再看 showNextToastLocked，

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

再看 scheduleTimeoutLocked 源码：

```java
NotificationManagerService
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

总结一下，scheduleTimeoutLocked 的主要工作就是调用 callback.hide 方法隐藏 Toast，然后从 mToastQueue 中移除刚才显示过的 Toast，最后，如果 mToastQueue 不为空，就继续显示下一个 Toast。

我们发现，Toast 的显示和隐藏都是和 callback 有关。那么 callback 到底是什么，callback 是 Toast 的一个静态内部类 TN。

```java
private static class TN extends ITransientNotification.Stub {
}
```

可以看出，TN 是一个 Binder，那么调用 TN 的方法，就是跨进程了。

它有两个方法 show 和 hide，分别对应 Toast 的显示和隐藏。show 和 hide 方法都是跨进程的。

```java
@Override
public void show() {
    if (localLOGV) Log.v(TAG, "SHOW: " + this);
    mHandler.post(mShow);
}

/**
 * schedule handleHide into the right thread
 */
@Override
public void hide() {
    if (localLOGV) Log.v(TAG, "HIDE: " + this);
    mHandler.post(mHide);
}
```

再看 mShow 和 mHide，可以发现它们两个都是 Runnable

```java
final Runnable mShow = new Runnable() {
    @Override
    public void run() {
        handleShow();
    }
};

final Runnable mHide = new Runnable() {
    @Override
    public void run() {
        handleHide();
        // Don't do this in handleHide() because it is also invoked by handleShow()
        mNextView = null;
    }
};
```

再看 handleShow 和 handleHide 方法：

```java
public void handleShow() {
	......
    mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
    ......
    mWM.addView(mView, mParams);
}

public void handleHide() {
	......
    if (mView.getParent() != null) {
        if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
        mWM.removeView(mView);
    }
}
```

可以发现，handleShow 和 handleHide 才是真正完成显示和隐藏 Toast 的地方。
TN 的 handleShow 方法会将 Toast 视图添加到 Window 中，
TN 的 handleHide 方法会将 Toast 视图从 Window 中移除。

到这里，Toast 的显示过程已经分析完了。

再看下 Toast 的取消过程，

```java
toast.cancel();
```

源码：

```java
@Override
public void cancelToast(String pkg, ITransientNotification callback) {
    synchronized (mToastQueue) {
        cancelToastLocked(index);
    }
}

void cancelToastLocked(int index) {
	// 取出 ToastRecord
    ToastRecord record = mToastQueue.get(index);
	// 调用 callback，也就是 TN 的 hide 方法隐藏 Toast
    try {
        record.callback.hide();
    } catch (RemoteException e) {
        Slog.w(TAG, "Object died trying to hide notification " + record.callback
                + " in package " + record.pkg);
        // don't worry about this, we're about to remove it from
        // the list anyway
    }
	// 隐藏 Toast 之后，从 mToastQueue 中移除 ToastRecord
    mToastQueue.remove(index);
    keepProcessAliveLocked(record.pid);
    if (mToastQueue.size() > 0) {
        // Show the next one. If the callback fails, this will remove
        // it from the list, so don't assume that the list hasn't changed
        // after this point.
		// 如果 mToastQueue 不为空，说明还有 Toast 需要显示，就继续显示下一个 Toast
        showNextToastLocked();
    }
}
```

### 总结 ###

为什么 Toast 要先和 NMS（NotificationManagerService）跨进程通信，让 NMS 调用 TN 的 show 方法。为什么不直接让 Toast 调用 show 方法。

猜想：Toast 是一个全局的行为，不依赖于某个应用，也不依赖于某个 Activity，所以 Toast 的显示应该由系统来完成，而且 Toast 的源码也说明了这一点，mToastQueue 中最多能存在 50 个 Toast，只要是非系统应用，大家一起共用这 50 个 Toast，从这个角度看，Toast 也应该是由系统（NMS）来调用。