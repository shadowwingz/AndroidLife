![](art/how_service_start.jpg)

启动一个 Service 很简单，只需要调用

```java
Intent intent = new Intent(this, TestService.class);
startService(intent);
```

就可以启动一个 Service 了，但是，只会启动 Service 是不够滴，我们还要知道它是怎么启动的，做一个牛（Zhuang）逼的开发者。

和 [Activity 的工作过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_activity_start/how_activity_start.md) 中一样，我们也先提前讲下 Service 启动过程中涉及到的一些对象，以及它们的作用，避免看源码的时候一脸懵逼。

- ContextImpl，Context 实现类，我们平常调用 Context 的方法都是 ContextImpl 实现的，比如获取包名 `getPackageName`、获取资源 `getResources`。
- ActivityManagerService，简称 AMS，服务端对象，负责系统中所有 Service 的生命周期。
- ApplicationThread，用来实现 AMS 与 ActivityThread 之间的交互。在 AMS 需要管理相关应用程序中的 Service 的生命周期时，通过 Application 的代理对象与 ActivityThread 通讯。
- ApplicationThreadProxy，是 ApplicationThread 在服务端的代理，负责和客户端的 ApplicationThread 通讯，AMS 就是通过该代理与 ActivityThread 通信的。
- ActivityThread，App 的真正入口，与 AMS 一起配合，一起完成 Service 的管理工作，比如启动 Service。
- LoaderApk，用于保存一些和 APK 相关的信息（如资源文件位置、JNI 库位置等）。

先大致描述一下 Service 的启动过程：

> Activity 的启动过程，我们从 Context 的 startActivity 说起，先是 ContextImpl 的 startService，然后内部会通过 startServiceCommon 来尝试启动 Service，这是一个跨进程过程，它会调用 AMS 的 startService 方法，AMS 校验完 Service 的合法性后，会通过 ApplicationThread 回调到我们的进程，这也是一次跨进程过程，而 ApplicationThread 就是一个 Binder，回调逻辑是在我们进程的 Binder 线程池中完成，所以需要通过 Handler H 将其切回 UI 线程，启动 Service 对应的消息是 CREATE_SERVICE,它对应着 handleCreateService，在这个方法里面完成了 Service 的创建和启动。

我们把 Service 的启动过程和 [Activity 的工作过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_activity_start/how_activity_start.md) 比较一下，就会发现，它俩很像，这说明了虽然 Android 系统源码贼特么复杂，但依然是有迹可寻的。

我们来看 Activity 的 startService 方法：

```java

```