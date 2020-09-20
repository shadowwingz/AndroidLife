# Activity 的启动流程的一些问题

## 大致描述一下 Activity 的启动流程

Activity 的启动过程，我们从 Context 的 startActivity 说起，先是 ContextImpl 的 startActivity，然后内部会通过 Instrumentation 来尝试启动 Activity，这是一个跨进程过程，它会调用 AMS 的 startActivity 方法，当 AMS 校验完  Activity 的合法性后，会通过 ApplicationThread 回调到我们的进程，这也是一次跨进程过程，而 ApplicationThread 就是一个 Binder，回调逻辑是在我们进程的 Binder 线程池中完成，所以需要通过 Handler H 将其切回 UI 线程，第一个消息是 LAUNCH_ACTIVITY,它对应着 handleLaunchActivity，在这个方法里面完成了 Activity 的创建和启动。接着，在 Activity 的 onResume 中，Activity 的内容将开始渲染到 Window 上面，然后开始绘制直到我们可以看到。

## Activity 与 AMS 的交互

Activity 与 AMS 有两次交互，这两次交互是通过 Binder 跨进程通信完成的。

第一次交互是在 Activity 请求启动的时候，Activity 会跨进程调用 AMS 的 startActivity 方法，接着 AMS 会校验 Activity 的合法性。

第二次交互是在 AMS 校验完 Activity 的合法性之后，会跨进程调用 ApplicationThread，完成 Activity 的创建和启动。

## Activity 的参数和结果如何传递

当我们想从 ActivityA 跳转到 ActivityB 时，有时候需要携带参数，那参数是怎么传递过去的呢？

ActivityA 参数可以通过 `Bundle` 传递过去，Bundle 实现了 `Parcelable` 接口，因此可以跨进程传递。

如果从 ActivityB 回到 ActivityA 的时候需要回传结果，可以通过 `onActivityResult` 方法来传递，参数同样也是 `Bundle`。

ActivityB 可以通过 `setResult` 方法传入参数，ActivityA 就可以在 `onActivityResult` 中收到这个参数。

## Activity 传递的参数的大小限制，怎么突破这种限制

前面我们说过，Activity 传递参数是通过 Bundle 来传递的，那 Bundle 有没有大小限制呢？

当然有，大小限制为不到 1M，这个限制定义在 `frameworks/native/libs/binder/ProcessState.cpp` 中，一个叫 `BINDER_VM_SIZE` 的常量：

```cpp
ProcessState # BINDER_VM_SIZE

#define BINDER_VM_SIZE ((1 * 1024 * 1024) - sysconf(_SC_PAGE_SIZE) * 2)
```

[在线源码](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r30:frameworks/native/libs/binder/ProcessState.cpp;l=43)

_SC_PAGE_SIZE 是 Linux 内存页大小，一般是 4KB，所以大小限制准确来说是：

> 1M - 4KB * 2 = 1016 KB。

比 1M 略小。

1M 的话，用来传递数据当然是不够用了，如今随便一张图片很可能都不止 1M 了，那有没有方法可以突破这种限制呢？

### 同一进程内传递数据

如果是同一个进程内传递数据，我们可以专门定义一个类 DataHolder，用来持有我们要传递的数据：

```kotlin
object DataHolder {

  val data: MutableMap<String, WeakReference<Any>>

  init {
    data = HashMap()
  }

  fun save(id: String, obj: Any) {
    data[id] = WeakReference(obj)
  }

  fun retrieve(id: String): Any? {
    return data[id]?.get()
  }
}
```

- ActivityA 传递数据：`DataHolder.save("data", ByteArray(1024 * 1024)`
- ActivityB 取出数据：`val largeData = DataHolder.retrieve("data")`

### 跨进程传递大图



## Activity 如何实例化

Activity 是通过类加载器 ClassLoader 加载了需要的 Activity 类，并通过反射调用构造方法创建出了 Activity 对象。

相关代码在 ActivityThread 的 `performLaunchActivity` 方法中：

```java
ActivityThread # performLaunchActivity

activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);

Instrumentation # newActivity

public Activity newActivity(ClassLoader cl, String className,
            Intent intent)
        throws InstantiationException, IllegalAccessException,
        ClassNotFoundException {
    // 通过 ClassLoader.loadClass 来加载 Activity 类，
    // 并通过反射调用 Activity 的构造方法来创建 Activity 对象
    return (Activity)cl.loadClass(className).newInstance();
}
```

## Activity 生命周期如何流转



## Activity 的窗口如何展示

