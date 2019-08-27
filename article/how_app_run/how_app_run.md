### 程序入口

- 主线程的消息循环

程序入口是 ActivityThread 的 main 方法，在 main 方法中，会调用 prepareMainLooper 创建主线程的消息循环。

prepareMainLooper 方法会创建主线程的 Looper 和 MessageQueue，但是消息队列仅有这些还不够，还需要一个handler，handler是在哪里创建的呢？

ActivityThread有一个内部类 H，它的类型是  handler，它就是主线程的 handler，当 activitythread 对象被创建的时候，它也会被创建，到这里，主线程的消息循环就完成了。

消息循环完成了，接下来，就是要给消息队列发消息了。当我们启动一个 activity时，ams会调用ApplicationThread 的方法，Applicationthread再给H发消息，消息会发送到H关联的消息队列中，UI主线程会异步的从消息队列中取出消息并执行。

- 界面的显示

当activitythread收到activity启动的消息之后，便会创建activity对象