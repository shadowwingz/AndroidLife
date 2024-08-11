# Module 的使用场景

之前我们是通过修改User 的源码，在它的构造函数旁边加了 `@Inject` 注解，这样就可以告知 Dagger 怎么创建 User 对象。

但是如果 User 是三方库中的类，那我们就没办法加 `@Inject` 注解了，因为我们无法修改 User 的源码，

所以我们需要通过其它的方式来告知 Dagger 要怎么创建 User 对象，比如 Module。

# Module 使用

## 定义一个 ThirdPartyUser 类，用来模拟三方库中的类

```Kotlin
/**
 * 假设 ThirdPartyUser 是三方库中的类，我们无法修改它的源码
 */
class ThirdPartyUser
```



## 定义 Module，用来告知 Dagger 如何创建 ThirdPartyUser 对象

```Kotlin
import dagger.Module
import dagger.Provides
import dagger.hilt.migration.DisableInstallInCheck

/**
 * 由于我们是用的 Hilt，Hilt 会在编译期检查 Module 是否加了 InstallIn 注解，
 * 如果没有加，就会报错。我们这里是用了演示 Dagger 的用法，不需要 InstallIn 注解，
 * 所以用 @DisableInstallInCheck 来禁用这个注解检查。
 */
@DisableInstallInCheck
@Module
class NetModule {

    @Provides
    fun provideThirdPartyUser(): ThirdPartyUser {
        return ThirdPartyUser()
    }
}
```

## Module 细节说明

![](/三方框架/dagger/image/Module细节.png)

## 把 Module 装载到 Component 中

```Kotlin
// 这行代码让 Module 和 Component 建立关联
@Component(modules = [NetModule::class])
interface ApplicationComponent {
  fun inject(activity: MainActivity)
}
```

## 在 Activity 中注入 ThirdPartyUser

```Kotlin
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var thirdPartyUser: ThirdPartyUser

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        DaggerApplicationComponent.create().inject(this)
        Log.d("shadowwingz", "onCreate: thirdPartyUser: $thirdPartyUser")
    }
}
```