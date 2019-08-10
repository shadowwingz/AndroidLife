AndroidLife
==

<br>
<br>


- 源码解析
	- [AIDL 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/AIDL%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- [Binder 的使用及上层原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/Binder%20%E7%9A%84%E4%BD%BF%E7%94%A8%E5%8F%8A%E4%B8%8A%E5%B1%82%E5%8E%9F%E7%90%86.md)
	- [Toast 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/toast/toast.md)
    	- [Toast 显示过程涉及到的 Binder](https://github.com/shadowwingz/AndroidLife/blob/master/article/toast/toast.md#toast-%E6%98%BE%E7%A4%BA%E8%BF%87%E7%A8%8B%E6%B6%89%E5%8F%8A%E5%88%B0%E7%9A%84-binder)
		- [Toast 有个数限制吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/toast/toast.md#toast-%E6%9C%89%E4%B8%AA%E6%95%B0%E9%99%90%E5%88%B6%E5%90%97)
		- [Toast 可以自定义时长吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/toast/toast.md#toast-%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%AE%9A%E4%B9%89%E6%97%B6%E9%95%BF%E5%90%97)
	    - [子线程可以弹 Toast 吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/show_toast_in_thread/show_toast_in_thread.md)
	    	- [Looper.loop() 在 Toast 前面调用可以吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/show_toast_in_thread/show_toast_in_thread.md#looperloop-%E5%9C%A8-toast-%E5%89%8D%E9%9D%A2%E8%B0%83%E7%94%A8%E5%8F%AF%E4%BB%A5%E5%90%97)
	    	- [主线程也是先调用 Looper.loop，为什么主线程可以弹 Toast？](https://github.com/shadowwingz/AndroidLife/blob/master/article/show_toast_in_thread/show_toast_in_thread.md#%E4%B8%BB%E7%BA%BF%E7%A8%8B%E4%B9%9F%E6%98%AF%E5%85%88%E8%B0%83%E7%94%A8-looperloop%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%BB%E7%BA%BF%E7%A8%8B%E5%8F%AF%E4%BB%A5%E5%BC%B9-toast)
	- [AlertDialog 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/AlertDialog%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- [ThreadLocal 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/ThreadLocal%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- [消息机制源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md)
	    - [Handler、Looper、MessageQueue 分别运行在什么线程，为什么](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#handlerloopermessagequeue-%E5%88%86%E5%88%AB%E8%BF%90%E8%A1%8C%E5%9C%A8%E4%BB%80%E4%B9%88%E7%BA%BF%E7%A8%8B%E4%B8%BA%E4%BB%80%E4%B9%88)
	    - [Handler、Looper、MessageQueue 各有几个，为什么](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#handlerloopermessagequeue-%E5%90%84%E6%9C%89%E5%87%A0%E4%B8%AA%E4%B8%BA%E4%BB%80%E4%B9%88)
	    - [Handler、Looper、MessageQueue 是怎么关联的](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#handlerloopermessagequeue-%E6%98%AF%E6%80%8E%E4%B9%88%E5%85%B3%E8%81%94%E7%9A%84)
	    - [Handler 发送的消息可以插队执行吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#handler-%E5%8F%91%E9%80%81%E7%9A%84%E6%B6%88%E6%81%AF%E5%8F%AF%E4%BB%A5%E6%8F%92%E9%98%9F%E6%89%A7%E8%A1%8C%E5%90%97)
	    - [主线程的 Handler 怎么发消息到子线程的 Looper](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#%E4%B8%BB%E7%BA%BF%E7%A8%8B%E7%9A%84-handler-%E6%80%8E%E4%B9%88%E5%8F%91%E6%B6%88%E6%81%AF%E5%88%B0%E5%AD%90%E7%BA%BF%E7%A8%8B%E7%9A%84-looper)
	    - [如果 Activity 要退出了，但是 MessageQueue 中的消息还没执行完怎么办？](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#%E5%A6%82%E6%9E%9C-activity-%E8%A6%81%E9%80%80%E5%87%BA%E4%BA%86%E4%BD%86%E6%98%AF-messagequeue-%E4%B8%AD%E7%9A%84%E6%B6%88%E6%81%AF%E8%BF%98%E6%B2%A1%E6%89%A7%E8%A1%8C%E5%AE%8C%E6%80%8E%E4%B9%88%E5%8A%9E)
	    - [消息队列怎么退出？各有什么区别？](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_messagequeue_quit/how_messagequeue_quit.md)
	    - [消息队列退出后，还可以用 Handler 发消息吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E9%80%80%E5%87%BA%E5%90%8E%E8%BF%98%E5%8F%AF%E4%BB%A5%E7%94%A8-handler-%E5%8F%91%E6%B6%88%E6%81%AF%E5%90%97)
	    - [消息队列退出后，消息队列里的消息还会执行吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler/handler.md#%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E9%80%80%E5%87%BA%E5%90%8E%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97%E9%87%8C%E7%9A%84%E6%B6%88%E6%81%AF%E8%BF%98%E4%BC%9A%E6%89%A7%E8%A1%8C%E5%90%97)
	    - [Hanlder 使用不当导致内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler_memory_leak/handler_memory_leak.md#hanlder-%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E5%AF%BC%E8%87%B4%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)
	    - [Handler 是怎么造成内存泄漏的](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler_memory_leak/handler_memory_leak.md#handler-%E6%98%AF%E6%80%8E%E4%B9%88%E9%80%A0%E6%88%90%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E7%9A%84)
	    - [为什么 Handler 发送的延时消息没有执行，会导致内存泄漏？](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler_memory_leak/handler_memory_leak.md#%E4%B8%BA%E4%BB%80%E4%B9%88-handler-%E5%8F%91%E9%80%81%E7%9A%84%E5%BB%B6%E6%97%B6%E6%B6%88%E6%81%AF%E6%B2%A1%E6%9C%89%E6%89%A7%E8%A1%8C%E4%BC%9A%E5%AF%BC%E8%87%B4%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)
	    - [怎么用代码检测内存泄漏？](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler_memory_leak/handler_memory_leak.md#%E6%80%8E%E4%B9%88%E7%94%A8%E4%BB%A3%E7%A0%81%E6%A3%80%E6%B5%8B%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)
	    - [避免内存泄漏的 Handler 写法](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler_memory_leak/handler_memory_leak.md#%E9%81%BF%E5%85%8D%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E7%9A%84-handler-%E5%86%99%E6%B3%95)
	    - [Android中为什么主线程不会因为 Looper.loop() 里的死循环卡死？（未完成）]()
	    - [手写消息机制](https://github.com/shadowwingz/AndroidLife/blob/master/article/handwriting_handler/handwriting_handler.md)
	- [AsyncTask 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/asynctask/asynctask.md)
	    - [如果在两个 Activity 中各 new 一个 AsyncTask，任务会排队执行吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/asynctask/asynctask.md#%E5%A6%82%E6%9E%9C%E5%9C%A8%E4%B8%A4%E4%B8%AA-activity-%E4%B8%AD%E5%90%84-new-%E4%B8%80%E4%B8%AA-asynctask%E4%BB%BB%E5%8A%A1%E4%BC%9A%E6%8E%92%E9%98%9F%E6%89%A7%E8%A1%8C%E5%90%97)
	    - [AsyncTask 执行的任务个数有限制吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/asynctask/asynctask.md#asynctask-%E6%89%A7%E8%A1%8C%E7%9A%84%E4%BB%BB%E5%8A%A1%E4%B8%AA%E6%95%B0%E6%9C%89%E9%99%90%E5%88%B6%E5%90%97)
	    - [AsyncTask 是串行还是并行，怎么实现并行](https://github.com/shadowwingz/AndroidLife/blob/master/article/asynctask/asynctask.md#asynctask-%E6%98%AF%E4%B8%B2%E8%A1%8C%E8%BF%98%E6%98%AF%E5%B9%B6%E8%A1%8C%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E5%B9%B6%E8%A1%8C)
	    - [AsyncTask 可以自定义线程池吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/asynctask/asynctask.md#asynctask-%E5%8F%AF%E4%BB%A5%E8%87%AA%E5%AE%9A%E4%B9%89%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%90%97)
	    - [AsyncTask 可以在子线程被调用吗？](https://github.com/shadowwingz/AndroidLife/blob/master/article/asynctask/asynctask.md#asynctask-%E5%8F%AF%E4%BB%A5%E5%9C%A8%E5%AD%90%E7%BA%BF%E7%A8%8B%E8%A2%AB%E8%B0%83%E7%94%A8%E5%90%97)
	- [子线程中能否更新 UI，为什么？]()
	- [IntentService 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/intentservice/intentservice.md)
    	- [连续启动 3 次 IntentService，任务是按顺序执行吗？为什么？](https://github.com/shadowwingz/AndroidLife/blob/master/article/intentservice/intentservice.md#%E8%BF%9E%E7%BB%AD%E5%90%AF%E5%8A%A8-3-%E6%AC%A1-intentservice%E4%BB%BB%E5%8A%A1%E6%98%AF%E6%8C%89%E9%A1%BA%E5%BA%8F%E6%89%A7%E8%A1%8C%E5%90%97%E4%B8%BA%E4%BB%80%E4%B9%88)
	- [事件分发源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- [滑动冲突解决方式之外部拦截法](https://github.com/shadowwingz/AndroidLife/blob/master/article/%E6%BB%91%E5%8A%A8%E5%86%B2%E7%AA%81%E8%A7%A3%E5%86%B3%E6%96%B9%E5%BC%8F%E4%B9%8B%E5%A4%96%E9%83%A8%E6%8B%A6%E6%88%AA%E6%B3%95.md)
	- [滑动冲突解决方式之内部拦截法](https://github.com/shadowwingz/AndroidLife/blob/master/article/%E6%BB%91%E5%8A%A8%E5%86%B2%E7%AA%81%E8%A7%A3%E5%86%B3%E6%96%B9%E5%BC%8F%E4%B9%8B%E5%86%85%E9%83%A8%E6%8B%A6%E6%88%AA%E6%B3%95.md)
	- [MeasureSpec 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/MeasureSpec%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- View 的工作流程
	    - [View 的 measure 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/View%20%E7%9A%84%20measure%20%E5%8E%9F%E7%90%86.md)
	    - [View 的 layout 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/View%20%E7%9A%84%20layout%20%E5%8E%9F%E7%90%86.md)
	    - [View 的 draw 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/View%20%E7%9A%84%20draw%20%E5%8E%9F%E7%90%86.md)
	- ViewGroup 的工作流程
	    - [ViewGroup 的 measure 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/ViewGroup%20%E7%9A%84%20measure%20%E5%8E%9F%E7%90%86.md)
	    - [LinearLayout 的 measure 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/LinearLayout%20%E7%9A%84%20measure%20%E5%8E%9F%E7%90%86.md)
	    - [LinearLayout 的 layout 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/LinearLayout%20%E7%9A%84%20layout%20%E5%8E%9F%E7%90%86.md)
	    - [RelativeLayout 的 measure 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/RelativeLayout%20%E7%9A%84%20measure%20%E5%8E%9F%E7%90%86.md)
	    - [RelativeLayout 的 layout 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/RelativeLayout%20%E7%9A%84%20layout%20%E5%8E%9F%E7%90%86.md)
	    - [LinearLayout 和 RelativeLayout 的 draw 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/LinearLayout%20%E5%92%8C%20RelativeLayout%20%E7%9A%84%20draw%20%E5%8E%9F%E7%90%86.md)
	- [RemoteViews 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/RemoteViews%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- Window 的内部机制
	    - [Window 的添加过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/Window%20%E7%9A%84%E6%B7%BB%E5%8A%A0%E8%BF%87%E7%A8%8B.md)
	    - Window 的删除过程
	    - Window 的更新过程
	- [LayoutInflater 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/LayoutInflater%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- [setContentView 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/setContentView%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
	- [LayoutParams 解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/LayoutParams%E8%A7%A3%E6%9E%90.md)
	- 四大组件工作过程
	    - [Activity 的工作过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_activity_start/how_activity_start.md)
	        - [Activity 启动过程中涉及到的 Binder](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_activity_start/how_activity_start.md#activity-%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B%E4%B8%AD%E6%B6%89%E5%8F%8A%E5%88%B0%E7%9A%84-binder)
	        - [Activity 启动过程中有几次跨进程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_activity_start/how_activity_start.md#activity-%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B%E4%B8%AD%E6%9C%89%E5%87%A0%E6%AC%A1%E8%B7%A8%E8%BF%9B%E7%A8%8B)
	    - Service 的工作过程
	        - [Service 的启动过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_service_start/how_service_start.md)
    	        - [Service 启动过程中涉及到的 Binder](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_service_start/how_service_start.md#service-%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B%E4%B8%AD%E6%B6%89%E5%8F%8A%E5%88%B0%E7%9A%84-binder)
	        - [Service 的绑定过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_service_bind/how_service_bind.md)
    	        - [Service 绑定过程中涉及到的 Binder](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_service_bind/how_service_bind.md#service-%E7%BB%91%E5%AE%9A%E8%BF%87%E7%A8%8B%E4%B8%AD%E6%B6%89%E5%8F%8A%E5%88%B0%E7%9A%84-binder)
	    - BroadcastReceiver 的工作过程
	        - [广播的注册过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_broadcast_register/how_broadcast_register.md)
	        - 广播的发送和接收过程
	        	- [普通广播的发送和接收过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_normal_broadcast_send/how_normal_broadcast_send.md)
	    - ContentProvider 的工作过程
    	    - [ContentProvider 的启动过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_contentprovider_start/how_contentprovider_start.md)
        	    - [ContentProvider 支持跨进程吗？为什么？](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_contentprovider_start/how_contentprovider_start.md#contentprovider-%E6%94%AF%E6%8C%81%E8%B7%A8%E8%BF%9B%E7%A8%8B%E5%90%97%E4%B8%BA%E4%BB%80%E4%B9%88)
        	    - [ContentProvider 中涉及到的 Binder](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_contentprovider_start/how_contentprovider_start.md#contentprovider-%E4%B8%AD%E6%B6%89%E5%8F%8A%E5%88%B0%E7%9A%84-binder)
	- [ListView 源码解析(未完成)](https://github.com/shadowwingz/AndroidLife/blob/master/article/listview/listview.md)
	- [Notification 的显示过程](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_notification_show/how_notification_show.md)

- 开源框架源码解析
    - [EventBus 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/eventbus/eventbus.md)
        - [EventBus 注册订阅者源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/eventbus/eventbus_register.md)
        - [EventBus 发送事件源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/eventbus/eventbus_post.md)
        - [EventBus 解除注册源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/eventbus/eventbus_unregister.md)
        - [EventBus 源码 + 注释](https://github.com/shadowwingz/EventBus)
    - [Retrofit 源码解析(未完成)]()
    - [RxJava 源码解析(未完成)]()
    - [Glide 源码解析(未完成)]()


- 自定义 View
    - [自定义 View 优惠券](https://github.com/shadowwingz/AndroidLife/blob/master/article/coupon_display_view/coupon_display_view.md)
    - [自定义 View 仪表盘](https://github.com/shadowwingz/AndroidLife/blob/master/article/dashboard/dashboard.md)


- 性能优化
    - [使用 Looper 检测应用中的 UI 卡顿](https://github.com/shadowwingz/AndroidLife/blob/master/article/use_looper_to_detect_ui/use_looper_to_detect_ui.md)
    - [使用 TraceView 检测应用中的卡顿](https://github.com/shadowwingz/AndroidLife/blob/master/article/trace_view/trace_view.md)
    - [ANR 是怎么发生的](https://github.com/shadowwingz/AndroidLife/blob/master/article/how_app_anr/how_app_anr.md)
    - [ANR 日志分析](https://github.com/shadowwingz/AndroidLife/blob/master/article/anr_analysis/anr_analysis.md)
    - [Android 绘制性能分析（未完成）]()
    - [内存泄漏的检查与分析](https://github.com/shadowwingz/AndroidLife/blob/master/article/memory_leak/memory_leak.md)
        - [什么是内存泄漏？](https://github.com/shadowwingz/AndroidLife/blob/master/article/memory_leak/memory_leak.md#%E4%BB%80%E4%B9%88%E6%98%AF%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)
        - [内存泄漏的危害](https://github.com/shadowwingz/AndroidLife/blob/master/article/memory_leak/memory_leak.md#%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E7%9A%84%E5%8D%B1%E5%AE%B3) 
        - [怎么判断对象可以回收](https://github.com/shadowwingz/AndroidLife/blob/master/article/memory_leak/memory_leak.md#%E6%80%8E%E4%B9%88%E5%88%A4%E6%96%AD%E5%AF%B9%E8%B1%A1%E5%8F%AF%E4%BB%A5%E5%9B%9E%E6%94%B6)
        - [怎么发现内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/memory_leak/memory_leak.md#%E6%80%8E%E4%B9%88%E5%8F%91%E7%8E%B0%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)
    - 内存泄漏的 N 种姿势
        - [Handler 延时消息造成的内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler_memory_leak/handler_memory_leak.md)
            - [怎样避免 Handler 延时消息内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/handler_memory_leak/handler_memory_leak.md#%E9%81%BF%E5%85%8D%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E7%9A%84-handler-%E5%86%99%E6%B3%95)
        - [不正确使用单例模式造成的内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/singleton_memory_leak/singleton_memory_leak.md)
            - [怎样避免单例模式内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/singleton_memory_leak/singleton_memory_leak.md#%E6%80%8E%E6%A0%B7%E9%81%BF%E5%85%8D%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)
        - [多线程造成的内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/thread_memory_leak/thread_memory_leak.md)
        	- [怎样避免多线程引起的内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/thread_memory_leak/thread_memory_leak.md#%E9%81%BF%E5%85%8D%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%BC%95%E8%B5%B7%E7%9A%84%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F)
        - [源码](https://github.com/shadowwingz/AndroidLifeDemo/tree/master/AndroidLifeDemo/app/src/main/java/com/shadowwingz/androidlifedemo/memoryleakdemo)
    - [使用 MAT 定位内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/mat_usage/mat_usage.md)
    - [使用 LeakCanary 定位内存泄漏](https://github.com/shadowwingz/AndroidLife/blob/master/article/leakcanary_usage/leakcanary_usage.md)
    - 缓存策略之 LruCache
        - LruCache 源码解析


- 细节知识点
    - `setResult` 调用时机
    - [compileSdkVersion、minSdkVersion、targetSdkVersion 含义](https://github.com/shadowwingz/AndroidLife/blob/master/article/compileSdkVersion%E3%80%81minSdkVersion%E3%80%81targetSdkVersion%20%E5%90%AB%E4%B9%89.md)
