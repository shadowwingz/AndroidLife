### [Demo 地址](https://github.com/shadowwingz/AndroidLifeDemo/tree/master/AndroidLifeDemo/app/src/main/java/com/shadowwingz/androidlifedemo/binderdemo) ###

Binder 的重要性就不用我多说了，Binder 的难度就更不用我多说了，所以即使《Android 开发艺术探索》里的 Binder 章节已经看了不少遍了，但是还是决定写篇文章，好好捋一捋 Binder。

行了，废话不多说，我们先来建个项目，然后新建个 binderdemo 文件夹，用于存放 Binder 相关的文件。我们先在 binderdemo 文件夹下新建个 Book.java，代码如下：

```java
/**
 * Created by shadowwingz on 2019-01-12 21:27
 * <p>
 * 实体类，实现了 Parcelable 接口
 */
public class Book implements Parcelable {

    public int mBookId;
    public String mBookName;

    public Book(int bookId, String bookName) {
        mBookId = bookId;
        mBookName = bookName;
    }

    protected Book(Parcel in) {
        mBookId = in.readInt();
        mBookName = in.readString();
    }

    public static final Creator<Book> CREATOR = new Creator<Book>() {
        @Override
        public Book createFromParcel(Parcel in) {
            return new Book(in);
        }

        @Override
        public Book[] newArray(int size) {
            return new Book[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(mBookId);
        dest.writeString(mBookName);
    }
}
```

Book 是个实体类，里面有两个字段，一个 `mBookId`，一个 `mBookName`，然后剩下的一堆代码都是为了实现 Parcelable 接口而写的模板代码。就不多说了。后面有必要的话会单独写篇文章来分析下 Parcelable 接口。总之，我们现在只需要知道 Book 实体类里有两个字段，然后 Book 又实现了 Parcelable 接口，所以支持跨进程传输（IPC，全名 Inter-Process Communication）。

等等，什么是跨进程传输，为什么实现了 Parcelable 接口就可以跨进程传输了？

嗯，说到跨进程，其实我懂得也不是很多，我先打个比方，张三家和李四家离的比较远，张三想给李四寄本书，李四不方便过来拿，所以张三就要找个快递，然后通过快递把书寄给李四，寄快递当然要打包了，然后李四收到快递之后，要拆快递，拿出张三寄的书。

在这个例子中，张三和李四就是两个进程，书就是要跨进程传输的对象，书打包快递就是 Book 类实现了 Parcelable 接口，只有把书打包成了快递包装，才能寄快递。李四收到快递后，要拆快递才能拿到书。同样，收到数据的进程要按照 Parcelable 接口规则来解析数据，才能取出相应的字段（mBookId 和 mBookName）。

行了，我们继续写代码。

刚刚新建了 Book.java，然后这个实体类也实现了 Parcelable 接口，但是还不够，我们还要在 AIDL 文件里实现这个实体类，而且这个 AIDL 文件名还要和实体类名字相同。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/%E5%88%9B%E5%BB%BAaidl%E6%96%87%E4%BB%B6.png"/>
</p>

然后我们会发现一件很尴尬的事，Android Studio 报错（2.3.2 版本），接口名字不能重复。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/%E6%97%A0%E6%B3%95%E5%88%9B%E5%BB%BABook%E7%9A%84aidl%E6%96%87%E4%BB%B6.png"/>
</p>

好吧，那我们先创建个 IBook.aidl 文件，创建好了再把名字改回来。

最终的 Book.aidl：

```java
// IBook.aidl
package com.shadowwingz.androidlifedemo.binderdemo;

// 自定义的实体类需要在 aidl 文件中声明
parcelable Book;
```

在 Book.aidl 中，我们只做了一件事，就是声明 Book 实体类。

说到这里，我们要解释一下 AIDL 文件是什么，以及为什么要创建 AIDL 文件？

Android Studio 会根据我们编写的 AIDL 文件来生成对应的 Binder 代码的 Java 文件，既然是生成 Java 文件，那我们也可以自己写 Java 文件，这样就不用再写 AIDL 文件了？没错，我们完全可以抛开 AIDL 文件直接写 Binder，说白了，AIDL 的作用就是方便系统为我们生成代码。不过，在这篇文章中，我们还是要用 AIDL 文件，来分析系统为我们生成的代码，进而分析 Binder 的上层原理。

接着，我们再创建一个 IBookManager.aidl，IBookManager.aidl 中我们定义两个操作，一个是获取图书列表，一个是添加图书。之所以不用普通的接口（interface），而是用 AIDL ，原因是为了跨进程。

IBookManager.aidl 代码如下：

```java
// IBookManager.aidl
package com.shadowwingz.androidlifedemo.aidl;

import com.shadowwingz.androidlifedemo.aidl.Book;

interface IBookManager {
    List<Book> getBookList();
    void addBook(in Book book);
}
```

AIDL 里定义了两个方法，其中 `addBook` 方法的形参类型 Book 前面有个 `in`，这个叫参数的方向。这里暂时不说太多，后面有时间再说。

我们在 AIDL 文件中写代码时，Android Studio 不会帮我们自动导包，另外，尽管 Book.aidl 和 IBookManager.aidl 在同一个文件夹下，但是还是需要导入 Book。所以，啥也别说了，手写导包，首先把 Book.aidl 导入。然后定义两个方法。

好了，AIDL 文件已经准备好了，接下来，Rebuild 一下项目。我们会发现在 `build/generated/source/aidl/debug/包名` 目录多了一个 IBookManager.java 文件。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/aidl%E6%96%87%E4%BB%B6%E7%94%9F%E6%88%90Binder%E7%B1%BB.png"/>
</p>

我们把这个 IBookManager.java 文件拷贝到 binderdemo 文件夹中，和 Book.java 放在一起。然后把刚刚写的 Book.aidl 和 IBookManager.aidl 从系统生成的 aidl 文件夹中转移到 binderdemo 文件夹中。最终目录结构如下：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/%E7%A7%BB%E5%8A%A8aidl%E6%96%87%E4%BB%B6.png"/>
</p>

之所以要移动 AIDL 文件到我们自己创建的 binderdemo 目录，是因为 IBookManager.java 已经被我们移动到我们自己创建的 binderdemo 目录了，项目一编译，就会再根据 AIDL 文件生成一个 IBookManager.java。这时就会报错（重复文件）。所以，为了让 AIDL 文件不受编译影响，我们把 AIDL 文件转移到我们自己创建的 binderdemo 文件夹中。

到这里，AIDL 的准备工作就差不多做好了，接下来，我们要使用 AIDL 进行进程间通信。也就是，刚刚我们打的比方，寄快递。

首先，先介绍一下流程，这里假设进程 A 和进程 B 进行进程间通信，进程 A 会跨进程调用进程 B 的方法：

1. 服务端

这里的服务端并不是指服务器，而是指进程 B，也就是被调用的一端（这点在我刚学 Binder 的时候让我困惑了很久），并且，客户端和服务端也是相对的，进程 A 调用了进程 B 的方法，那么此时进程 A 就是客户端，进程 B 就是服务端，然后进程 B 又调用了进程 A 的方法，那么此时进程 B 就是客户端，进程 A 就是服务端。在本例中，我们只讨论进程 A 调用进程 B 的方法。

服务端首先要创建一个 Service 用来监听客户端（进程 A）的连接请求，然后创建一个 AIDL 文件（刚刚我们创建的 IBookManager.aidl），将暴露给客户端的接口（getBookList 方法和 addBook 方法）在这个 AIDL 文件中声明，最后在 Service 中实现这个 AIDL 接口。

2. 客户端

客户端首先要绑定服务端的 Service，绑定成功后，将服务端返回的 Binder 对象转换成 AIDL 接口所属的类型，就可以调用 AIDL 中的方法了，也就是远程调用进程 B 的方法。

3. AIDL 接口的创建

刚刚我们已经创建好了，就是 IBookManager.aidl。

4. 远程服务端（进程 B） Service 的实现

我们先创建一个 Service，叫做 BookManagerService，代码如下：

```java
public class BookManagerService extends Service {

    // 因为 Binder 的接口方法会在线程池中执行，所以需要用到支持并发读写的 CopyOnWriteArrayList
    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();

    // 实现 Binder 的接口方法，当客户端远程调用 getBookList 和 addBook 时，
    // 这两个方法会在 Binder 线程池中执行。
    private Binder mBinder = new IBookManager.Stub() {
        @Override
        public List<Book> getBookList() throws RemoteException {
            LogUtil.d("服务端被调用，返回图书列表： " + mBookList + " 当前进程：" + Util.getProcessName(BookManagerService.this));
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            LogUtil.d("服务端被调用，添加图书： " + book + " 当前进程：" + Util.getProcessName(BookManagerService.this));
            mBookList.add(book);
        }
    };

    public BookManagerService() {
    }

    @Override
    public void onCreate() {
        super.onCreate();
        // 添加两本图书信息
        mBookList.add(new Book(1, "Android"));
        mBookList.add(new Book(2, "Ios"));
    }

    @Override
    public IBinder onBind(Intent intent) {
        // 返回 Binder 对象
        return mBinder;
    }
}
```

上面是一个服务端 Service 的简单实现，因为我们是要跨进程通信的，所以 Service 需要运行在一个独立的进程，所以我们修改一下 AndroidManifest.xml：

```xml
<service
    android:name=".binderdemo.BookManagerService"
    android:process=":remote">
</service>
```

然后，我们新建一个 BookManagerActivity，来作为客户端。在 BookManagerActivity 里，我们要绑定远程服务，绑定成功后将服务端返回的 Binder 对象转换为 AIDL 接口，然后就可以通过这个接口去调用服务端的方法了。

```java
public class BookManagerActivity extends AppCompatActivity {

    private ServiceConnection mConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager = IBookManager.Stub.asInterface(service);
            try {
                List<Book> bookList = bookManager.getBookList();
                LogUtil.d("客户端添加图书之前，查询到的图书列表 " + bookList +
                        " 当前进程：" + Util.getProcessName(BookManagerActivity.this));

                Book newBook = new Book(3, "Android 开发艺术探索");
                LogUtil.d("客户端添加图书 " + newBook +
                        " 当前进程：" + Util.getProcessName(BookManagerActivity.this));
                bookManager.addBook(newBook);

                List<Book> newList = bookManager.getBookList();
                LogUtil.d("客户端添加图书之后，查询到的图书列表 " + newList +
                        " 当前进程：" + Util.getProcessName(BookManagerActivity.this));
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_book_manager);
        Intent intent = new Intent(this, BookManagerService.class);
        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
    }
}
```

我们在进行每个跨进程调用时，都打印了方法执行时所在的进程名。

启动一下 app，打印日志如下：

```
服务端被调用，返回图书列表： [Book{mBookId=1, mBookName='Android'}, Book{mBookId=2, mBookName='Ios'}] 当前进程：com.shadowwingz.androidlifedemo:remote
客户端添加图书之前，查询到的图书列表 [Book{mBookId=1, mBookName='Android'}, Book{mBookId=2, mBookName='Ios'}] 当前进程：com.shadowwingz.androidlifedemo
客户端添加图书 Book{mBookId=3, mBookName='Android 开发艺术探索'} 当前进程：com.shadowwingz.androidlifedemo
服务端被调用，添加图书： Book{mBookId=3, mBookName='Android 开发艺术探索'} 当前进程：com.shadowwingz.androidlifedemo:remote
服务端被调用，返回图书列表： [Book{mBookId=1, mBookName='Android'}, Book{mBookId=2, mBookName='Ios'}, Book{mBookId=3, mBookName='Android 开发艺术探索'}] 当前进程：com.shadowwingz.androidlifedemo:remote
客户端添加图书之后，查询到的图书列表 [Book{mBookId=1, mBookName='Android'}, Book{mBookId=2, mBookName='Ios'}, Book{mBookId=3, mBookName='Android 开发艺术探索'}] 当前进程：com.shadowwingz.androidlifedemo
```

可以发现，客户端和服务端的代码的确是执行在两个进程中。到这里，我们已经完整的使用 AIDL 完成了 IPC 过程。

接下来，我们深入研究一下，在 IPC 的过程中，经历了那些流程。

我们就从 bindService 开始，当我们调用下面的代码时：

```java
Intent intent = new Intent(this, BookManagerService.class);
bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
```
s
BookManagerService 被启动，`mBookList` 被实例化， `mBinder` 也被创建出来，BookManagerService 的 `onCreate` 方法也会被调用：

```java
private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();

private Binder mBinder = new IBookManager.Stub() {
    @Override
    public List<Book> getBookList() throws RemoteException {
        LogUtil.d("服务端被调用，返回图书列表： " + mBookList + " 当前进程：" + Util.getProcessName(BookManagerService.this));
        return mBookList;
    }

    @Override
    public void addBook(Book book) throws RemoteException {
        LogUtil.d("服务端被调用，添加图书： " + book + " 当前进程：" + Util.getProcessName(BookManagerService.this));
        mBookList.add(book);
    }
};


@Override
public void onCreate() {
    super.onCreate();
    mBookList.add(new Book(1, "Android"));
    mBookList.add(new Book(2, "Ios"));
}

@Override
public IBinder onBind(Intent intent) {
    return mBinder;
}
```

这里重点说下 mBinder，mBinder 是 `IBookManager.Stub` 的实现类的实例，我们看下 `IBookManager.Stub`：

```java
public interface IBookManager extends android.os.IInterface {
    public static abstract class Stub extends android.os.Binder implements IBookManager {
        ......
    }

    public java.util.List<Book> getBookList() throws android.os.RemoteException;

    public void addBook(Book book) throws android.os.RemoteException;
}
```

可以看到，Stub 是个抽象类，它实现了 IBookManager 的接口，但是并没有实现 IBookManager 接口中的方法，IBookManager 接口中的方法是由开发者去实现，也就是我们去实现。

关于 IBookManager.java 的讲解，可以看下 [AIDL 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/AIDL%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)。

接着，由于我们是以 bindService 的方式启动的 Service，所以 BookManagerService 的 onBind 会被调用，mBinder 被返回：

```java
@Override
public IBinder onBind(Intent intent) {
    return mBinder;
}
```

被返回到哪里呢？被返回到 BookManagerActivity 的 ServiceConnection 的 onServiceConnected 回调方法中。

```java
private ServiceConnection mConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        // service 就是 BookManagerService 返回的 mBinder
        IBookManager bookManager = IBookManager.Stub.asInterface(service);
        ......
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {

    }
```

BookManagerActivity 收到 mBinder 之后，会通过 `IBookManager.Stub.asInterface` 方法把 Binder 转换为接口。为什么要转换呢？

首先，是因为要调用接口的方法，我们定义的 `getBookList` 方法和 `addBook` 方法是写在 IBookManager 接口中，不转换成 IBookManager 就没办法调用接口中方法了。
第二，asInterface 方法会根据客户端和服务端是否在同一进程而返回不同的接口对象，如果是在同一进程，就直接返回服务端的 Stub 本身，如果不在同一进程，就返回系统封装后的 `Stub.proxy` 对象。

```java
if (((iin != null) && (iin instanceof IBookManager))) {
    return ((IBookManager) iin);
}
return new Proxy(obj);
```

由于我们的客户端和服务端不在同一进程，所以 asInterface 方法返回的是 `Stub.proxy` 对象。

```java
private static class Proxy implements IBookManager {
}
```

好了，到这里，我们就拿到 IBookManager 接口了，接着，我们调用 `getBookList` 方法去查询图书列表：

```java
List<Book> bookList = bookManager.getBookList();
```

在我们调用了 `bookManager.getBookList()` 方法之后，在内部会调用 Proxy 的 getBookList 方法，getBookList 方法运行在客户端，接着创建 `getBookList` 方法所需要的输入型 Parcel 对象 _data，输出型 Parcel 对象 _reply 和返回值对象 _result。

_data 就是我们调用 getBookList 方法时传入的参数，当然，这里我们没有传入参数，所以 _data 就是一个空的 Parcel 数据。

_reply 就是我们调用 `getBookList` 方法后，方法返回的结果，也是 Parcel 数据。

_result 就是我们从 Parcel 数据中解析出的我们真正想要的数据 List。

接着，会调用 transact 方法发起 RPC（远程过程调用）请求，同时当前线程挂起（简单理解，就是先暂停着）。我们看下 transact 方法：

```java
IBinder # transact

// code 是一个数字，表示客户端要执行的操作
// flags 我们这里不分析
public boolean transact(int code, Parcel data, Parcel reply, int flags)
    throws RemoteException;
```

data 和 reply 我们刚刚说了，这里就不多说了。这里重点说下 code，首先我们可以看到，code 是一个 int 型数字，它表示了客户端要执行的操作，为什么这么说呢？当客户端调用服务端的方法时，会把 code 传递过去，然后服务端会去寻找它有没有这个 code，如果找到了 code，就去执行相应的方法，也就是客户端想调用的方法。在这个例子中，code 就是 `TRANSACTION_getBookList`。从这里我们也可以分析出，客户端调用服务端的方法也不是直接就跨进程调用了，而是要根据 code 来寻找有没有要调用的方法，有就调用，没有就没办法调用了。

我们继续分析，当调用了 transact 后，这个调用请求会通过系统底层封装后交给 onTransact 方法处理，onTransact 是运行在服务端的 Binder 线程池（就是个线程池，别想多了），也就是说，现在方法已经执行到服务端了。

```java
@Override
public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int
        flags) throws android.os.RemoteException {
    switch (code) {
        case INTERFACE_TRANSACTION: {
            reply.writeString(DESCRIPTOR);
            return true;
        }
        // 刚刚客户端传递过来的 code 是 TRANSACTION_getBookList，
        // 服务端查找自己这边有没有 TRANSACTION_getBookList 这个 code。
        case TRANSACTION_getBookList: {
            data.enforceInterface(DESCRIPTOR);
            java.util.List<Book> _result =
                    this.getBookList();
            reply.writeNoException();
            reply.writeTypedList(_result);
            return true;
        }
        case TRANSACTION_addBook: {
            data.enforceInterface(DESCRIPTOR);
            Book _arg0;
            if ((0 != data.readInt())) {
                _arg0 = Book.CREATOR
                        .createFromParcel(data);
            } else {
                _arg0 = null;
            }
            this.addBook(_arg0);
            reply.writeNoException();
            return true;
        }
    }
    return super.onTransact(code, data, reply, flags);
}

static final int TRANSACTION_getBookList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
static final int TRANSACTION_addBook = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
```

刚刚我们说了，当我们调用 `bookManager.getBookList()` 方法之后，服务端就根据 `code` 去查找客户端要调用的方法是什么，这里 `getBookList` 的 code 是 `TRANSACTION_getBookList`，于是就走到了 `onTransact` 的 `TRANSACTION_getBookList` case 中，于是就调用了 `this.getBookList()` 方法，也就是 BookManagerService 中实现的 Binder 方法，刚刚说过，`onTransact` 方法是运行在服务端的线程池中，这也给了我们启示，跨进程调用的方法是可以耗时的，不需要我们手动使用线程池，我们继续看BookManagerService 实现的 Binder 方法：

```java
private Binder mBinder = new IBookManager.Stub() {
    @Override
    public List<Book> getBookList() throws RemoteException {
        LogUtil.d("服务端被调用，返回图书列表： " + mBookList + " 当前进程：" + Util.getProcessName(BookManagerService.this));
        return mBookList;
    }

    ......
};
```

到这里，终于完成了跨进程调用，但是还没完，因为这只是调用了服务端的 `getBookList` 方法，获取到了图书列表，接下来还要把这个图书列表返回到客户端才算完事。

OK，那我们继续分析，还是看下 `onTransact` 方法里的 `TRANSACTION_getBookList` 的 case：

```java
java.util.List<Book> _result = this.getBookList();
reply.writeNoException();
reply.writeTypedList(_result);
return true;
```

可以看到，返回的图书列表被赋值给了 `_result`，然后调用 `reply.writeTypedList(_result)` 方法把 `_result` 封装到了 `_reply` 中。为什么要封装呢？List 本身是不支持跨进程传输的，所以我们要把 List 转换成支持跨进程传输的类型，也就是 Parcel 类型。也就是上文的打比方，寄快递，只有把书装到快递袋里，快递才能帮你运输。

`reply.writeTypedList(_result)` 方法执行完后，`reply` 中就有图书列表了。然后执行下一句代码，`return true`，到这里，RPC 过程结束，回到客户端刚刚挂起的方法的地方，注意，从这里开始，就从服务端又回到了客户端：

```java
@Override
public java.util.List<Book> getBookList()
        throws android.os.RemoteException {
    android.os.Parcel _data = android.os.Parcel.obtain();
    android.os.Parcel _reply = android.os.Parcel.obtain();
    java.util.List<Book> _result;
    try {
        _data.writeInterfaceToken(DESCRIPTOR);
        // 客户端挂起
        mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
        _reply.readException();
        // _reply 中就是刚刚获取的图书列表，因为 _reply 是 Parcel 类型的数据，
        // 我们没法直接用，所以要从 _reply 中把图书列表取出来，存入 _result 中
        _result = _reply.createTypedArrayList(Book.CREATOR);
    } finally {
        _reply.recycle();
        _data.recycle();
    }
    return _result;
```

可以看到，客户端也拿到了服务端返回的数据，服务端返回的数据是在 `_reply` 中，然后我们从 `_reply` 中取出图书列表。到这里，Proxy 的 `getBookList` 方法就返回图书列表了。接着，在 BookManagerActivity 的 `onServiceConnected` 中，`bookManager.getBookList` 也返回了图书列表。

到这里，一次完整的 IPC 过程就分析完了。我们分析的是 `getBookList` 方法，其实 `addBook` 也差不多，这里就不分析了。