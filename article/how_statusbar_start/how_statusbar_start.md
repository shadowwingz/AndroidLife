状态栏运行在一个叫做 SystemUIService 的 Service 中，所以状态栏的启动也就是 SystemUIService 的启动。

SystemUIService 的启动从 SystemServer 开始，在 SystemServer 的 run 方法中，会启动各种核心系统服务，启动完核心系统服务之后，就会调用 `startOtherServices` 方法来启动其他系统服务。

```java
SystemServer # run

private void run() {
    // Start services.
    try {
        ......
        // 启动核心系统服务
        startCoreServices();
        // 启动其他系统服务，其中包括 SystemUIService
        startOtherServices();
    } catch (Throwable ex) {
        ......
    }
}
```

再看看 `startOtherServices` 方法：

```java
private void startOtherServices() {
    ......
    mActivityManagerService.systemReady(new Runnable() {
        @Override
        public void run() {
            ......
            try {
                startSystemUi(context);
            } catch (Throwable e) {
                reportWtf("starting System UI", e);
            }
            ......
        }
    }
    ......
}
```

`startOtherServices` 方法中，调用了 `mActivityManagerService.systemReady` 方法，并传入了一个 Runnable 参数，当 `systemReady` 方法执行完后，就会回调 这个 Runnable 任务，在 Runnable 任务中，又调用了 `startSystemUi(context)` 方法：

```
static final void startSystemUi(Context context) {
    Intent intent = new Intent();
    intent.setComponent(new ComponentName("com.android.systemui",
                "com.android.systemui.SystemUIService"));
    //Slog.d(TAG, "Starting service: " + intent);
    context.startServiceAsUser(intent, UserHandle.OWNER);
}
```

在 `startSystemUi` 中，调用 `startServiceAsUser` 方法启动了 `com.android.systemui.SystemUIService`，到这里，SystemUIService 的启动就完成了。

