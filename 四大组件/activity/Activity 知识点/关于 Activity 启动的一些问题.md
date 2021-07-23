# 关于 Activity 启动的一些问题

<!-- TOC -->

- [大致描述一下 Activity 的启动流程](#%E5%A4%A7%E8%87%B4%E6%8F%8F%E8%BF%B0%E4%B8%80%E4%B8%8B-activity-%E7%9A%84%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B)
- [Activity 与 AMS 的交互](#activity-%E4%B8%8E-ams-%E7%9A%84%E4%BA%A4%E4%BA%92)
- [Activity 的参数和结果如何传递](#activity-%E7%9A%84%E5%8F%82%E6%95%B0%E5%92%8C%E7%BB%93%E6%9E%9C%E5%A6%82%E4%BD%95%E4%BC%A0%E9%80%92)
- [Activity 传递的参数的大小限制，怎么突破这种限制](#activity-%E4%BC%A0%E9%80%92%E7%9A%84%E5%8F%82%E6%95%B0%E7%9A%84%E5%A4%A7%E5%B0%8F%E9%99%90%E5%88%B6%E6%80%8E%E4%B9%88%E7%AA%81%E7%A0%B4%E8%BF%99%E7%A7%8D%E9%99%90%E5%88%B6)
    - [同一进程内传递数据](#%E5%90%8C%E4%B8%80%E8%BF%9B%E7%A8%8B%E5%86%85%E4%BC%A0%E9%80%92%E6%95%B0%E6%8D%AE)
    - [跨进程传递大图](#%E8%B7%A8%E8%BF%9B%E7%A8%8B%E4%BC%A0%E9%80%92%E5%A4%A7%E5%9B%BE)
- [Activity 如何实例化](#activity-%E5%A6%82%E4%BD%95%E5%AE%9E%E4%BE%8B%E5%8C%96)
- [Activity 生命周期如何流转](#activity-%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%A6%82%E4%BD%95%E6%B5%81%E8%BD%AC)
- [Activity 的窗口如何展示](#activity-%E7%9A%84%E7%AA%97%E5%8F%A3%E5%A6%82%E4%BD%95%E5%B1%95%E7%A4%BA)
- [禁止外部三方应用启动 Activity](#%E7%A6%81%E6%AD%A2%E5%A4%96%E9%83%A8%E4%B8%89%E6%96%B9%E5%BA%94%E7%94%A8%E5%90%AF%E5%8A%A8-activity)
- [可被外部启动的 Activity 需要注意拒绝服务漏洞](#%E5%8F%AF%E8%A2%AB%E5%A4%96%E9%83%A8%E5%90%AF%E5%8A%A8%E7%9A%84-activity-%E9%9C%80%E8%A6%81%E6%B3%A8%E6%84%8F%E6%8B%92%E7%BB%9D%E6%9C%8D%E5%8A%A1%E6%BC%8F%E6%B4%9E)

<!-- /TOC -->

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


## 禁止外部三方应用启动 Activity

在 `AndroidManifest.xml` 的 Activity 节点中，将 `exported` 属性设置为 false。

```java
<activity android:name=".MainActivity" android:exported="false">
</activity>
```

如果我们自己的 App 确实需要将某个 Activity 暴露出去，但又不想随便一个三方 App 都可以调，那么可以用 permission 来进一步控制。

比如我们有一个应用 A，想要调起应用 B 的 Activity。

我们可以先在 B 的 `AndroidManifest.xml` 中定义一个权限，然后给对应的 Activity 设置这个权限：

```xml
<permission android:name="com.example.MYPERMISSON" />

<activity android:name=".MainActivity"
    android:permission="com.example.MYPERMISSON" >
</activity>
```

然后在 A 的 `AndroidManifest.xml` 中声明一下这个权限：

```xml
<uses-permission android:name="com.example.MYPERMISSON" />
```

这样的话，A 就可以访问 B 的 MainActivity 了，其他 App 如果没有声明权限，就无法访问 B 的 MainActivity。

## 可被外部启动的 Activity 需要注意拒绝服务漏洞

我们的 Activity 在被三方 App 启动的时候，三方 App 可能会传递数据过来，数据是通过 Bundle 传输。三方 App 通过 `bundle.putxxx()` 传递数据，我们通过 `bundle.getxxx()` 取出数据，如果三方 App 传递过来一个我们 App 中不存在的实体类，那么我们就会解析失败。

比如三方 App 中有一个实体类 ModelA，

```java
class ModelA implements Serializable {

}
```

而我们自己的 App 中并没有定义这么一个实体类 ModelA，那么解析就会出错。

这种情况下，我们的 App 应该在解析的时候加上 `try catch`。