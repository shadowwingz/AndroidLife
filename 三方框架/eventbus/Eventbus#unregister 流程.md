# Eventbus#unregister 流程

首先，我们看看 EventBus 中定义的几个变量的含义：

```
/**
 * 以
 *
 * onReceive(Event event) {}
 *
 * EventBus.getDefault().register(MainActivity.this);
 *
 * 为例
 *
 * key : Event，
 * value 是一个 List，List 中的对象都是 Subscription，
 * 而 Subscription 中又封装了 MainActivity 和 onReceive，也就是订阅者信息。
 * 
 * 同一个事件，可能有多个订阅者订阅
 */
private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;

/**
 * key : 订阅者，比如 MainActivity
 * value : Event
 * 
 * 一个订阅者可能订阅了多种类型的事件
 */
private final Map<Object, List<Class<?>>> typesBySubscriber;
```

接着我们来看 EventBus 是怎么解除注册的

以 

```
EventBus.getDefault().register(MainActivity.this)

onReceive(Event event) {}
``` 

为例：

EventBus 解除注册，使用的是 unregister 方法，我们来看下这个方法：

```java
public synchronized void unregister(Object subscriber) {
    // 获取订阅者订阅的所有事件类型
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    if (subscribedTypes != null) {
        for (Class<?> eventType : subscribedTypes) {
            // 对单个事件类型解除注册
            unsubscribeByEventType(subscriber, eventType);
        }
        // 将订阅者订阅的所有事件类型解除注册之后，从 typesBySubscriber 中移除订阅者
        typesBySubscriber.remove(subscriber);
    } else {
        Log.w(TAG, "Subscriber to unregister was not registered before: " + subscriber.getClass());
    }
}
```

首先，获取订阅者订阅的所有事件类型，订阅者就是 MainActivity，获取 MainActivity 订阅的所有事件类型，然后遍历所有事件，调用 unsubscribeByEventType 解除注册：

```java
private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
    // 获取订阅事件对应的所有订阅者信息
    List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions != null) {
        int size = subscriptions.size();
        for (int i = 0; i < size; i++) {
            // 遍历所有的订阅者信息，找到要解除注册的那一个订阅者，然后把它移除掉。
            Subscription subscription = subscriptions.get(i);
            if (subscription.subscriber == subscriber) {
                subscription.active = false;
                subscriptions.remove(i);
                i--;
                size--;
            }
        }
    }
}
```

首先，我们要找到 Event 对应的所有订阅者，也就是所有订阅了 Event 对象的订阅者。这些订阅者既有我们要解除注册的对象 MainActivity，也有其他的 Activity 或者 Fragment。接着我们遍历所有订阅者，找到我们要解除注册的订阅者，将它的 active 状态置为 false，再把这个订阅者从订阅者列表中移除掉。

到这里，EventBus 的解除注册流程我们就分析完了，我们最后再总结一下：

> EventBus 会从 typesBySubscriber 和 subscriptions 中分别移除订阅者以及相关信息。