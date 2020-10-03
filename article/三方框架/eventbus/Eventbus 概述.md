# Eventbus 概述

EventBus 想必每个 Android 开发都不陌生，它是一个 Android 事件发布/订阅框架，通过解耦发布者和订阅者简化 Android 事件传递。传统的事件传递包括：Handler、BroadcastReceiver、Interface 回调，而 EventBus 与它们相比，代码更加简洁，使用起来也更加简单。

EventBus 的源码分析，我觉得应该从使用入手。回想下我们是怎么使用 EventBus，首先注册一下订阅者。一般我们是在 Activity 里的 onCreate 中来注册：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    EventBus.getDefault().register(this);
}
```

接着我们在 Activity 的 onDestory 中解除注册：

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    EventBus.getDefault().unregister(this);
}
```

接着我们要在特定的方法里来接收 EventBus 发送的事件，什么是特定的方法呢？就是被 EventBus 的注解修饰的方法：

```java
class Event {

}

@Subscribe(threadMode = ThreadMode.MAIN)
public void onReceive(Event event) {

}
```

`@Subscribe(threadMode = ThreadMode.MAIN)` 表示 onReceive 方法在主线程中执行，当我们调用 `EventBus.getDefault().post(new Event())` 发送一个事件时，onReceive 最终会在主线程中被回调，我们发送的 event 对象也会被传递到 onReceive 的 event 形参里。

EventBus 的基本用法到这里我们基本就过了一遍，那么我们的源码分析主要分析以下几个方面：

- `EventBus.getDefault().register(this)` 这句代码，EventBus 是怎么注册订阅者的

[EventBus 注册订阅者源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/eventbus/eventbus_register.md)

- `EventBus.getDefault().post(new Event())` 这句代码，EventBus 是怎么发送事件的

[EventBus 发送事件源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/eventbus/eventbus_post.md)

- `onReceive` 方法，是怎么接收到事件的

[EventBus 接收事件源码解析]()