在我们日常开发的时候，通常会在内存中存储一些数据。比如我们开发一个联系人列表，我们需要定义一个 `List<String>` 来存储列表联系人的姓名。如果此时 app 一旋转，我们发现，联系人列表的数据没了，界面瞬间变的光秃秃的。这样的体验会很不好。

那么，怎么解决这个问题呢？

我们可以思考一下，联系人列表的数据之所以没了，是因为我们的数据是存在 Activity 里，屏幕旋转，导致 Activity 销毁后重建。Activity 会销毁，我们的联系人列表数据 `List<String>` 自然也被回收了。

那么是不是说，如果我们可以定义一个类 `DataHelper`，`DataHelper` 专门用来持有联系人列表数据，屏幕旋转的时候，Activity 会被销毁，但是 `DataHelper` 不会被销毁，因此 `DataHelper` 中存储的数据也不会丢失。

看样子问题好像已经解决了，但是我们再想一想，`DataHelper` 中的数据应该什么时候清除呢？理想的状态是，Activity 启动的时候获取到联系人列表数据，然后存储到 `DataHelper` 中，我们按返回键，Activity 退出，这个时候再清除 `DataHelper` 中存储的联系人列表数据，或者干脆直接把 `DataHelper` 销毁。但是这样开发者就会稍微辛苦一点，每写一个 Activity/Fragment 就要注意在 onDestroy 中销毁对应的 `DataHelper。`

那有没有更简单的方法呢？我就想要一个能让我放心储存数据的类，我只管往这个类里丢数据，页面一销毁，这个类就会自动跟着被销毁，不用开发者再操心手动销毁。

答案是有的，就是我们接下来要说明的 ViewModel。

我们新建一个类 `DataHelperViewModel`，继承自 ViewModel。

这个 `DataHelperViewModel` 就可以解决刚刚我们遇到的问题。

想储存数据又不想手动清除？没问题，你随意存，Acticity 一旦退出，`DatahelperViewModel` 的数据会自动销毁。

屏幕旋转数据会丢失？不用担心，屏幕旋转，Activity 会重建，但是 DatahelperViewModel 不会重建，所以我们储存的数据并不会丢。

而且 `DataHelperViewModel` 还有一个 `onClear` 的方法，这个方法会在 Activity 退出的时候被回调，类似 Activity 的 `onDestroy` 回调。所以我们可以在这个方法执行一些资源回收操作。比如取消还未执行完成的任务。