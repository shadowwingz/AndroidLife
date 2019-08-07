Handler 算是面试必问的知识点了，Looper、MessageQueue 相信不少童鞋也能说个一二三，那么我们来挑战点更难的：手写 Handler。

要想手写 Handler，我们也要手写 Looper 和 MessageQueue。那我们就要思考下，Handler、Looper 和 MessageQueue 中都有些什么属性和方法，它们之间又是怎么协作的？

#### Handler 里面有什么？ ####

回忆一下我们使用 Handler，都是调用 Handler 的 sendMessage 方法来发送一个消息，接着这个消息会被投递到消息队列，也就是 MessageQueue 中，所以我们知道，Handler 中肯定有 MessageQueue 的引用。

#### Looper 里面有什么？ ####