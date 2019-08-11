![](art/1.jpg)

[点击查看大图](https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/article/how_notification_show/art/1.jpg)

通知是 App 中很常见的一个功能，我们来研究下通知是怎么显示出现的。首先，我们要知道，通知是运行在 System UI 中的，System UI 是 Android 系统内置的应用，包括系统通知栏、状态栏，等待。我们这里分析的就是通知栏。通知栏是运行在 System UI 进程的。

通知是调用 `notify` 方法显示出来的，`notify` 方法会调用 NotificationManager 的 `notify` 方法：

```java
NotificationManager # notify

public void notify(int id, Notification notification)
{
    notify(null, id, notification);
}

public void notify(String tag, int id, Notification notification)
{
    ......
    // 获取 NMS 的实例
    INotificationManager service = getService();
    ......
    try {
        // 远程调用 NMS 的 enqueueNotificationWithTag 方法
        service.enqueueNotificationWithTag(pkg, mContext.getOpPackageName(), tag, id,
                stripped, idOut, UserHandle.myUserId());
        ......
    } catch (RemoteException e) {
    }
}
```

在 `notify` 方法中，会远程调用 NMS（NotificationManagerService）的 `enqueueNotificationWithTag` 方法，代码会执行到 system server 进程。

```java
NotificationManagerService # enqueueNotificationWithTag

@Override
public void enqueueNotificationWithTag(String pkg, String opPkg, String tag, int id,
        Notification notification, int[] idOut, int userId) throws RemoteException {
    enqueueNotificationInternal(pkg, opPkg, Binder.getCallingUid(),
            Binder.getCallingPid(), tag, id, notification, idOut, userId);
}

NotificationManagerService # enqueueNotificationInternal

void enqueueNotificationInternal(final String pkg, final String opPkg, final int callingUid,
            final int callingPid, final String tag, final int id, final Notification notification,
            int[] idOut, int incomingUserId) {
    ......

    mHandler.post(new Runnable() {
        @Override
        public void run() {

            synchronized (mNotificationList) {
                ......

                if (notification.icon != 0) {
                    StatusBarNotification oldSbn = (old != null) ? old.sbn : null;
                    mListeners.notifyPostedLocked(n, oldSbn);
                }

                ......
            }
        }
    });

    ......
}
```

在 `enqueueNotificationWithTag` 中，会调用 `enqueueNotificationInternal` 方法，接着会调用 NotificationListeners 的 `notifyPostedLocked` 方法，

```java
NotificationListeners # notifyPostedLocked

public void notifyPostedLocked(StatusBarNotification sbn, StatusBarNotification oldSbn) {
    ......

    for (final ManagedServiceInfo info : mServices) {
        ......

        mHandler.post(new Runnable() {
            @Override
            public void run() {
                notifyPosted(info, sbnToPost, update);
            }
        });
    }
}

NotificationListeners # notifyPosted

private void notifyPosted(final ManagedServiceInfo info,
                final StatusBarNotification sbn, NotificationRankingUpdate rankingUpdate) {
    ......
    try {
        listener.onNotificationPosted(sbnHolder, rankingUpdate);
    } catch (RemoteException ex) {
        Log.e(TAG, "unable to notify listener (posted): " + listener, ex);
    }
}
```

在 `notifyPostedLocked` 方法中，最终会在 mHandler 关联的主线程中回调 INotificationListener 的 `onNotificationPosted` 方法，INotificationListener 对应的实现类是 `INotificationListenerWrapper`：

```java
INotificationListenerWrapper # onNotificationPosted

private class INotificationListenerWrapper extends INotificationListener.Stub {
    @Override
    public void onNotificationPosted(IStatusBarNotificationHolder sbnHolder,
            NotificationRankingUpdate update) {
        ......
        synchronized (mWrapper) {
            ......
            try {
                NotificationListenerService.this.onNotificationPosted(sbn, mRankingMap);
            } catch (Throwable t) {
                Log.w(TAG, "Error running onNotificationPosted", t);
            }
        }
    }
}

NMS # onNotificationPosted

public void onNotificationPosted(StatusBarNotification sbn, RankingMap rankingMap) {
    onNotificationPosted(sbn);
}

public void onNotificationPosted(StatusBarNotification sbn) {
    // optional
}
```

可以看到，INotificationListenerWrapper 是继承自 `INotificationListener.Stub` 的，因此可以判断出 INotificationListener 是一个 Binder，在 `onNotificationPosted` 中，调用了 NMS 的 `onNotificationPosted` 方法，接着，通知会传递给我们的通知栏，也就是 PhoneStatusBar，但是会先传到 BaseStatusBar 的 `onNotificationPosted` 中：

```java
BaseStatusBar # onNotificationPosted

@Override
public void onNotificationPosted(final StatusBarNotification sbn,
        final RankingMap rankingMap) {
    ......
    mHandler.post(new Runnable() {
        @Override
        public void run() {
            ......
            if (isUpdate) {
                // 更新通知
                updateNotification(sbn, rankingMap);
            } else {
                // 显示一条新通知
                addNotification(sbn, rankingMap);
            }
        }
    });
}

PhoneStatusBar # addNotification

@Override
public void addNotification(StatusBarNotification notification, RankingMap ranking) {
    ......
    addNotificationViews(shadeEntry, ranking);
    ......
}
```

在 `onNotificationPosted` 中，会调用 `addNotification` 来显示通知，到这里，一个通知，就从 App 进程，传递到 System Server 进程，又被传递到 System UI 进程，最后在 `addNotification` 方法中显示出来。