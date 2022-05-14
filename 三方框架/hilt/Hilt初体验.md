<!-- TOC -->

- [添加 Hilt 依赖](#添加-hilt-依赖)
- [开始使用 Hilt](#开始使用-hilt)
- [使用 Hilt 实现字段注入](#使用-hilt-实现字段注入)
- [Hilt 工作流程概述](#hilt-工作流程概述)

<!-- /TOC -->

## 添加 Hilt 依赖

使用 Hilt 前首先需要加入依赖：

项目的 `build.gradle` 中：

```gradle
buildscript {
    ...
    ext.hilt_version = '2.28-alpha'
    dependencies {
        ...
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```

`app/build.gradle` 中：

```gradle
...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

dependencies {
    ...
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```

## 开始使用 Hilt

首先，我们需要自定义一个 Application，并给这个 Application 加上 `@HiltAndroidApp` 注解，有了这个注解，Hilt 才能起作用。

```kotlin
@HiltAndroidApp
class LogApplication : Application() {
    ...
}
```

## 使用 Hilt 实现字段注入

假如我们在 Fragment 中定义了一个 DateFormatter 字段：

```kotlin
class LogsFragment : Fragment() {

    lateinit var dateFormatter: DateFormatter
}
```

要使用 Hilt 实例化这个字段，我们需要几个步骤：

1. 给 LogsFragment 加上 `@AndroidEntryPoint` 注解，这个注解会告诉 Hilt，LogsFragment 中需要依赖注入。这样 Hilt 才会在 LogsFragment 中寻找需要被依赖注解的字段并给它自动赋值。

```kotlin
@AndroidEntryPoint
class LogsFragment : Fragment() {
}
```

2. 对需要被依赖注入的变量，添加 `@Inject` 注解，这样 Hilt 才能找到哪些变量需要被依赖注入。

```kotlin
@Inject
lateinit var dateFormatter: DateFormatter
```

3. 给 DateFormatter 的构造方法加上 `@Inject` 注解，以告知 Hilt 如何初始化该字段。

```kotlin
class DateFormatter @Inject constructor() {
}
```

4. 如果有哪个 Activity 用到了这个 Fragment，也需要给对应的 Activity 加上 `@AndroidEntryPoint` 注解。

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
}
```

## Hilt 工作流程概述

1. 应用启动，LogApplication 会被执行，程序发现 LogApplication 有 `@HiltAndroidApp` 注解，于是 Hilt 开始工作。
2. 应用进入 MainActivity，MainActivity 中加载了 LogsFragment。而 LogsFragment 配置了 `@AndroidEntryPoint`。于是程序告知 Hilt，LogsFragment 中可能有字段进行依赖注入。
3. Hilt 开始查找 LogsFragment 中需要被依赖注入的字段，查找方式是看哪个字段有 `@Inject` 注解，于是找到了 `dateFormatter` 字段。
4. Hilt 发现 `dateFormatter` 字段的类型是 `DateFormatter`，于是去查找有没有可以创建 `DateFormatter` 对象的方式。查找方式是看 `DateFormatter` 这个类的构造方法有没有添加 `@Inject` 注解。
5. Hilt 发现 `DateFormatter` 构造器有 `@Inject` 注解，于是调用构造方法创建一个 DateFormatter 对象，并赋值给 LogsFragment 的 `dateFormatter` 字段。到这里，`dateFormatter` 的依赖注入就完成了。