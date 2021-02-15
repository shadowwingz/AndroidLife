App 启动分为 3 种：

- 冷启动
- 热启动
- 温启动

测量启动时间的 adb 指令：

```java
adb shell am start -W com.shadowwingz.wanandroid/com.shadowwingz.wanandroid.ui.ContainerActivity
```

打印结果：

```java
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.shadowwingz.wanandroid/.ui.ContainerActivity }
Status: ok
Activity: com.shadowwingz.wanandroid/.ui.ContainerActivity
ThisTime: 665
TotalTime: 665
WaitTime: 687
Complete
```

- ThisTime：最后一个 Activity 启动耗时
- TotalTime：所有 Activity 启动耗时
- WaitTime：AMS 启动 Activity 的总耗时

用 adb 命令可以方便的测出 Activity 的启动时间，但是这种方式仅限于线下使用，不能带到线上，比较线上没有 adb 环境。

那线上的话，可以这么测量呢？我们可以采用手动打点的方式来测量。

手动打点，就是使用 System.currentTimeMillis 来获取 Activity 启动开始和结束的时间点并相减，得到时间差，这个时间差就是 Activity 的启动时间。

那么 Activity 的启动开始和结束是在哪里呢？

说到 Activity 的启动开始时间点，这个没什么争议，就是 Activity 的 onCreate 方法。

那 Activity 的启动结束时间点呢？我们认为，当界面上控件被绘制出来的时候，就算 Activity 启动结束。这个时间点我们可以用 `view.viewTreeObserver.addOnPreDrawListener` 监听到控件被绘制出来。



### 怎么看哪里的代码耗时

- TraceView
- Systrace

