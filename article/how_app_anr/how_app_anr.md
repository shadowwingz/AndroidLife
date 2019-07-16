当我们使用 App 的时候，有时候可能会点击 App 没反应，然后过一会，就弹出一个弹框提示，提示用户，当前应用未响应，是继续等待还是强制关闭。

这种情况叫做 ANR，那么哪些情况下 App 会出现 ANR 呢？在 

#### Service 超时 ####

当前台 Service 的 onCreate 方法中，进行了耗时操作（超过 20 秒）：

```java
public class ForegroundService extends Service {
    public ForegroundService() {
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Intent i = new Intent(this, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(getApplication(), 0, i, 0);
        Notification notification = new Notification.Builder(
                getApplication()).setAutoCancel(true).setSmallIcon(R.mipmap.ic_launcher)
                .setTicker("前台Service启动").setContentTitle("前台Service运行中").
                setContentText("这是一个正在运行的前台Service").setWhen(System.currentTimeMillis())
                .setContentIntent(pendingIntent).build();
        startForeground(1, notification);
        SystemClock.sleep(25 * 1000);
        return super.onStartCommand(intent, flags, startId);
    }
}
```

上面的代码运行会报 ANR 错误。

当后台 Service 的 onCreate 方法中，进行了耗时操作（超过 200 秒）：

#### BroadcastReceiver 超时 ####

当广播发送给广播接收者时，接收者需要处理这个广播，也就是在 onReceive 方法中执行自己的逻辑，这个逻辑的执行时间是有限制的，对于前台广播，是 10 秒，对于后台广播，是 60 秒。超过这个时间就会 ANR。

前台广播就是发送广播的时候，给 intent 添加一个 flag：

```java
Intent mIntent = new Intent("android.intent.action.xxx");  
mIntent.addFlags(Intent.FLAG_RECEIVER_FOREGROUND);   
mContext.sendBroadcast(mIntent);    
```

#### ContentProvider 超时 ####

当我们使用 ContentProvider 时，一般是获取第三方 App 内部的数据，如果当我们使用 ContentProvider 获取数据的时候，第三方 App 还没有启动，这种情况下，第三方 App 提供数据的时间可能会超过 10 秒，如果超过 10 秒，就会 ANR。

#### 输入事件超时 ####

当 App 的主线程因为执行了耗时方法导致卡住的时候，我们点击 App，App 是没有反应的，如果超过 5 秒钟还没有响应，就会 ANR。