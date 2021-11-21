Gradle Plugin 想必大家并不陌生，当我们新建一个 Android 项目的时候，在 app 目录下的 `build.gradle` 文件中会有这么一行代码：

```
apply plugin: 'com.android.application'
```

这行代码的意思是，加载一个名字叫 `com.android.application` 的 gradle 插件。

那我们也来试试，自己写一个 gradle plugin。

在写 gradle plugin 之前，我们要先介绍点背景知识，`build.gradle` 文件是什么？是配置吗？

我们可以尝试跳转到 `apply` 方法的内部，可以看到 `apply` 是一个函数。

```java
public interface PluginAware {
  ......

  void apply(Map<String, ?> var1);

  ......
}

```

原来 `build.gradle` 是代码文件，和我们平常写的 java 代码文件 xxx.java 是一样的。只不过我们平常在使用 gradle 的时候，都是在配置东西，导致给我们的感觉 `build.gradle` 就是个配置文件。

既然是个代码文件，那说明我们可以直接在这个文件里写代码了。那我们就开始吧。

这里要先说明一下，我们在 `build.gradle` 中写的代码是用 groovy 语言写的，groovy 语言和 java 很像。

```
class FirstPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        println '我是 FirstPlugin'
    }
}
```

要写 gradle 插件，首先我们得定义一个类，也就是 class，这里的 class 不是 java 中的 class，而是 groovy 中的 class。

这个 class 实现了 Plugin 接口，这个也比较好理解，你实现了 Plugin 接口，才能被 gradle 文件视为一个 Plugin。

接着我们重写了 apply 文件，在 apply 方法中我们可以实现自己的逻辑，这里简单起见，我们只打印一行 log。

插件定义好了，我们要使用这个插件。使用插件的方法很简单，参考下刚刚那个 `apply plugin: 'com.android.application`，我们要使用自己的插件 FirstPlugin，那就把名字改为 FirstPlugin 就好了：

```
apply plugin: FirstPlugin
```

这样我们就完成了一个最简单的 Gradle Plugin。我们尝试运行一下这个 plugin，打开 Android Studio 的 Terminal，输入 `./gradlew`，观察一下打印的 log，在 log 中我们应该就可以看到我们定义的 log。

```
> Configure project :app
我是 FirstPlugin
```