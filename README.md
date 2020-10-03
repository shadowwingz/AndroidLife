AndroidLife
==

<br>
<br>

## 四大组件
### Activity

- [Activity 启动的大体流程（不涉及源码）](https://github.com/shadowwingz/AndroidLife/blob/master/article/activity/general_process/general_process.md)
- [Activity 启动流程（源码）](https://github.com/shadowwingz/AndroidLife/blob/master/article/activity/how_activity_start/how_activity_start.md)
- [setContentView 流程解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/activity/setContentView/setContentView.md)
- [onResume 里能获取 View 宽高吗？](https://github.com/shadowwingz/AndroidLife/blob/3da4ae4ae9fe8dc77758f0ead58930b7728f9c8f/article/activity/activity_start_questions/get_view_width_in_resume.md)
- [使用 View.post 获取 View 宽高](https://github.com/shadowwingz/AndroidLife/blob/master/article/activity/activity_start_questions/use_view_post_get_view_width.md)

### Service

- [Service 的启动流程]()
- [Service 的绑定流程]()

### BroadCast

- [广播的注册流程]()
- [普通广播的发送流程]()

### ContentProvider

- [ContentProvider 的启动过程]()

## 核心机制

### Handler

- [Handler 的工作流程]()
- [ThreadLocal 源码解析]()
- [Handler 细节之 IdleHandler]()
- [Handler 细节之内存屏障]()

### Binder

- [Binder 的使用及上层原理]()
- [AIDL 源码解析]()

## 常用组件

- [AsyncTask 工作原理]()
- [IntentService 工作原理]()
- [RemoteViews 源码解析]()

## Android 自带 UI 控件

### View & ViewGroup

- [MeasureSpec 源码解析]()
- [View 的 measure 原理]()
- [View 的 layout 原理]()
- [View 的 draw 原理]()
- [ViewGroup 的 measure 原理]()
- [LinearLayout 的 measure 原理]()
- [LinearLayout 的 layout 原理]()
- [LinearLayout 和 RelativeLayout 的 draw 原理]()
- [事件分发源码解析]()
- [用 Demo 体验事件分发]()
- [滑动冲突解决方式之内部拦截法]()
- [滑动冲突解决方式之外部拦截法]()
- [LayoutParams解析]()
- [LayoutInflater源码解析]()

### Toast

- [Toast 源码解析]()
- [子线程中可以弹 Toast 吗？]()

### 通知

- [通知的显示流程]()

### ListView

- [ListView 的工作原理]()

### AlertDialog

- [AlertDialog 源码解析]()

## 三方框架

### Eventbus

### Retrofit

### Glide

### Butterknife

### Rxjava

### Okhttp

## 性能优化

### 内存优化

- [图片内存优化]()
- [内存泄漏概述]()
- [ArrayList 引起的内存泄漏]()
- [Handler 使用不当导致内存泄漏]()
- [单例模式引起的内存泄漏]()
- [多线程引起的内存泄漏]()
- [使用 MAT 分析内存泄漏]()
- [使用 LeakCanary 分析内存泄漏]()
- [缓存策略之 LruCache]()

### 卡顿优化

- [使用 TraceView 定位卡顿]()
- [使用 Looper#setMessageLogging 检测卡顿]()

<br>
<br>

## UI 组件

### Toast

- [Toast 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/toast/toast.md)

- [非 UI 线程能否更新 UI（待完成）]()

<br>
<br>