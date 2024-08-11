- [前言](#前言)
- [gradle 依赖](#gradle-依赖)
- [使用最新版本 Hilt](#使用最新版本-hilt)
- [新建 User 类](#新建-user-类)
- [新建 Component 组件](#新建-component-组件)
- [在 Activity 中使用 Dagger](#在-activity-中使用-dagger)
- [总结](#总结)

# 前言

Dagger 两种注入方式

- 使用构造方法：在类的构造方法上使用`@Inject`注解，告知Dagger这是通过构造方法来创
建对象。
- 使用Dagger模块：使用`@Module`申明一个类为Dagger模块，并提供创建某个对象实例
的方法，然后使用`@Provides`或`@Binds`注解该方法。

# gradle 依赖

由于 Hilt 内部也依赖了 Dagger，所以我们直接依赖 Hilt 即可引入 Dagger。相当于我们使用的是 Hilt 中的 Dagger。

libs.versions.toml

```Toml
hilt = "2.44"

hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-android-compiler = { group = "com.google.dagger", name = "hilt-android-compiler", version.ref = "hilt" }

```



根目录 build.gradle.kts

```Kotlin
// Top-level build file where you can add configuration options common to all sub-projects/modules.
@Suppress("DSL_SCOPE_VIOLATION") // TODO: Remove once KTIJ-19369 is fixed
plugins {
    ......
    id("com.google.dagger.hilt.android") version "2.44" apply false
}
true // Needed to make the Suppress annotation work for the plugins block

allprojects {
    tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile>().configureEach {
        kotlinOptions {
            jvmTarget = JavaVersion.VERSION_1_8.toString()
        }
    }
}
```



app 目录下 build.gradle.kts

```Kotlin
@Suppress("DSL_SCOPE_VIOLATION") // TODO: Remove once KTIJ-19369 is fixed
plugins {
    ......
    kotlin("kapt")
    id("com.google.dagger.hilt.android")
}

dependencies {
    ......
    implementation(libs.hilt.android)
    kapt(libs.hilt.android.compiler)
}

```



# 使用最新版本 Hilt

使用编译报错，可以尝试使用最新版本的 Hilt，如 2.51。



# 新建 User 类

```Kotlin
import javax.inject.Inject
/**
 * 使用 @Inject 注解在构造方法上，
 * 就是告知 Dagger 可以通过构造方法来创建并获取到 User 的实例
 */
class User @Inject constructor()
```



# 新建 Component 组件

```Kotlin
import dagger.Component

// 使用了 @Component 注解，Dagger 会帮我们生成对应的 Dagger 类，
// 比如 ApplicationComponent -> DaggerApplicationComponent
@Component
interface ApplicationComponent {
    fun inject(activity: MainActivity)
}
```



# 在 Activity 中使用 Dagger

```Kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import javax.inject.Inject

class MainActivity : AppCompatActivity() {

    // 我们可以使用 @Inject 注解告知 Dagger，让 Dagger 帮我们构造一个 user 对象
    @Inject
    lateinit var user: User

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        DaggerApplicationComponent.create().inject(this)
        Log.d("shadowwingz", "onCreate: user: $user")
    }
}
```



DaggerApplicationComponent.create().inject(this) 这句代码很关键。

执行 inject(this) 后，Dagger 内部会做 3 件事：

1. Dagger 内部会保存 this，也就是 MainActivity，这是为了方便给 MainActivity 的 user 变量赋值（内部会执行 MainActivity.user = User()），这也是为什么 Dagger 注入的变量不能声明为 private，如果声明为 private，即使 Dagger 中保存了 MainActivity 对象，也没法访问 MainActivity.user 对象
2. Dagger 会扫描当前的 MainActivity 文件，如果发现有 Inject 注解的变量  user，Dagger 会帮我们构建一个 User 对象
3. Dagger 会把构建好的 User 对象赋值给 MainActivity

# 总结

![](/三方框架/dagger/image/使用Dagger快速实现依赖注入.png)

