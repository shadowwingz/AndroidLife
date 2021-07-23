DataBinding 除了可以让控件和值绑定起来，还可以使用 BindingMethod 来更便捷的实现一些效果。

比如，在我们的开发中，经常有加载图片的需求，代码大概是这样的：

```kotlin
Glide.with(context).load(imageUrl).into(imageView)
```

观察上面这段代码，我们发现，这段代码需要 3 个参数，imageView, context 和 imageUrl。
而 context 我们知道，可以通过 `imageView.getContext` 获取，所以我们实际上只需要 2 个参数：imageView 和 imageUrl。

那么，能不能写成这种形式呢：

```xml
<ImageView
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:imageUrl="url" />
```

我们知道 ImageView 是没有 imageUrl 这个属性，那我们能不能自定义一个 imageUrl 属性呢？这样我们直接给这个属性设置一个 imageUrl，就可以加载图片了，不用再写 Java/Kotlin 代码了。

答案是可以的。

BindingMethod 隆重登场~~~

如果用 BindingMethod 来实现，是这样的：

首先，我们需要新建一个类，类名不限，我们可以叫它 BindingAdapters，在 BindingAdapters 中，我们实现一个 loadImage 方法：

```kotlin
@BindingAdapter("app:imageUrl")
fun loadImage(view: ImageView, url: String) {
  Glide.with(view.context).load(url).into(view)
}
```

我们观察一下这个 loadImage 方法，发现它有一些特点：

1. 方法带有 `@BindingAdapter` 注解，注解中有一个 `app:imageUrl` 属性
2. 方法有 2 个参数，第一个是参数是 ImageView，第二个参数是 url
3. 方法的逻辑很简单，和我们刚刚写的加载图片的逻辑是完全一样的

我们挨个解释一下：

1. 我们要实现一个 BindingMethod 方法，就必须要加 `@BindingAdapter` 注解，这样编译器才能检测到这是一个 BindingMethod 方法。而 `app:imageUrl` 就是我们自定义的属性，这样我们才能在 xml 文件中通过 `app:imageUrl="url"` 来给一个 ImageView 设置 imageUrl 属性
2. BindingMethod 方法一般至少有 2 个参数，第一个参数一般是控件类型，这里是 ImageView，第二个参数一般是给控件使用的属性，这里是 url,表示我们要把 url 给 ImageView 使用。
3. 方法的逻辑就不用多说了，就是把逻辑代码从 Activity/Fragment 挪到 BindingMethod 中。

BindingMethod 定义好之后，我们再看看如何使用，这里我们把 url 放在 ViewModel 中：

```kotlin
class MyViewModel : ViewModel() {

  var url: String = "https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png"
}
```

然后在 xml 中引用这个 ViewModel 中的 url：

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:binding="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="data"
            type="com.shadowwingz.androidpractice.jetpack.databinding.MyViewModel" />

    </data>

    <!-- 省略部分代码 -->

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        app:imageUrl="@{data.url}" />
</layout>
```

这样我们就利用 BindingMethod 实现了加载图片的功能。等等，我们刚刚好像都没有用到 BindingAdapters 类，就是写 BindingMethod 的那个类。

是的，对于 BindingMethod 来说，类并不重要。重要的是 @BindingAdapter 注解，只要有这个注解，编译器就会知道这是个 BindingMethod，从而执行某些操作。所以 BindingMethod 所在的类并不重要，我们也不需要显式的去创建它的实例。