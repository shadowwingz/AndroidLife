### ViewModel 基本使用

我们这里使用 ViewModel 实现一个小 Demo，Demo 很简单，在屏幕上显示一个按钮，点击按钮，会开始计时，并实时把计时显示在屏幕上。

由于 ViewModel 一般都是和 LiveData 结合使用，所以本例中也会使用到 LiveData。

首先，我们新建一个类 LiveDataTimerViewModel，继承自 ViewModel。

```kotlin
class LiveDataTimerViewModel : ViewModel() {

  private val ONE_SECOND = 1000L

  /**
   * 计时
   */
  val mElapsedTime = MutableLiveData<Long>()

  /**
   * 初始时间
   */
  val mInitialTime by lazy { SystemClock.elapsedRealtime() }

  /**
   * 由于定时器 timer 是保存在 ViewModel 中，所以屏幕旋转的时候，timer 不会被回收，因此屏幕旋转不会影响计时。
   */
  private val timer = Timer()

  init {
    /**
     * 1 秒钟后启动计时，计时启动后，每隔 1 秒更新一次计时。
     */
    timer.scheduleAtFixedRate(
      object : TimerTask() {
        override fun run() {
          val newValue = (SystemClock.elapsedRealtime() - mInitialTime) / 1000
          mElapsedTime.postValue(newValue)
        }
      }, ONE_SECOND, ONE_SECOND
    )
  }

  /**
   * 页面退出时，会回调 ViewModel 的 onCleared 方法，在 onCleared 方法中，
   * 我们可以执行一些资源清理的操作，比如取消定时器。
   */
  override fun onCleared() {
    super.onCleared()
    timer.cancel()
  }
}
```

我们观察一下这个 ViewModel，我们可以发现 ViewModel 的逻辑主要分为 3 步：

1. 定义变量，这里我们定义了 `mElapsedTime` 变量来作为我们的计时，`Timer` 帮助我们实现定时器功能。
2. 更新变量，这里我们使用 Timer 来定时更新 `mElapsedTime` 变量
3. 回收变量，这里我们重写了 ViewModel 的 `onCleared` 方法，在 `onCleared` 方法中取消了定时器

变量的定义和更新都完成了，接下来我们需要让 Activity 监听到这个变量的更新。

```kotlin
class ChronoActivity2 : AppCompatActivity() {

  /**
   * 获取 ViewModel 的实例
   */
  val mLiveDataTimerViewModel by lazy {
    ViewModelProvider(this).get(LiveDataTimerViewModel::class.java)
  }

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_chrono2)

    subscribe()
  }

  private fun subscribe() {
    val elapsedTimeObserver = object : Observer<Long> {
      override fun onChanged(t: Long?) {
        val newText = resources.getString(R.string.seconds, t)
        timer_textview.text = newText
        LogUtil.d("更新计时器")
      }
    }

    /**
     * 监听 mElapsedTime 变量的变化
     */
    mLiveDataTimerViewModel.mElapsedTime.observe(this, elapsedTimeObserver)
  }
}
```

Activity 中的逻辑主要分为 2 步：

1. 实例化 ViewModel，刚刚我们定义了 ViewModel，我们自然也需要实例化 ViewModel，对于 ViewModel 的实例化，我们不能直接去 new，而是要使用 `ViewModelProvider(this).get(LiveDataTimerViewModel::class.java)` 的方式进行实例化。
2. 监听 ViewModel 中变量的变化，这里的监听是借助 LiveData 实现的。当我们在 ViewModel 中对 `mElapsedTime` 进行 `postValue` 操作时，在 Activity 中就可以监听到 `mElapsedTime` 值的变化，进而更新 UI。

这里我们就完成了 ViewModel 的基本使用。

我们再总结一下：

1. 新建一个类继承自 ViewModel，在 ViewModel 中我们定义和界面状态相关的变量，并且变量通常用 LiveData 修饰。
2. 在 ViewModel 中对变量进行 `setValue/postValue` 操作
3. 在 Activity 中实例化 ViewModel 对象
4. 对 ViewModel 中的变量进行监听，在监听回调中更新 UI