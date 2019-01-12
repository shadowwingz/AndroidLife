Binder 的重要性就不用我多说了，Binder 的难度就更不用我多说了，所以即使《Android 开发艺术探索》里的 Binder 章节已经看了不少遍了，但是还是决定写篇文章，好好捋一捋 Binder。

行了，废话不多说，我们先来建个 Demo，就叫 BinderDemo。新建个 aidl 文件夹，用于存放 Binder 相关的文件。我们先在 aidl 文件夹下新建个 Book.java，代码如下：

```java
package com.shadowwingz.binderdemo.aidl;

import android.os.Parcel;
import android.os.Parcelable;

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

刚刚新建了 Book.java，然后这个实体类也实现了 Parcelable 接口，但是还不够，我们还要在 aidl 文件里实现这个实体类，而且这个 aidl 文件名还要和实体类名字相同。

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
package com.shadowwingz.binderdemo.aidl;

// 自定义的实体类需要在 aidl 文件中声明
parcelable Book;
```

在 Book.aidl 中，我们只做了一件事，就是声明 Book 实体类。

在创建出 Book.aidl 文件后，我们惊奇的发现，现在居然有两个 aidl 文件夹。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/%E4%B8%A4%E4%B8%AAaidl%E6%96%87%E4%BB%B6%E5%A4%B9.png"/>
</p>

上面的 aidl 文件夹，是系统创建 aidl 文件时，顺便创建的。在这个文件夹里存放的 aidl 文件，在项目编译时会生成对应的 java 文件。

下面的 aidl 文件夹则是我们自己创建的文件夹。在这个文件夹里存放的 aidl 文件，在项目编译时不会生成对应的 java 文件。

接着，我们再创建一个 IBookManager.aidl，IBookManager.aidl 中我们定义两个操作，一个是获取图书列表，一个是添加图书。之所以不用普通的接口（interface），而是用 aidl ，原因是为了跨进程。

IBookManager.aidl 代码如下：

```java
// IBookManager.aidl
package com.shadowwingz.binderdemo.aidl;

import com.shadowwingz.binderdemo.aidl.Book;

interface IBookManager {
    List<Book> getBookList();
    void addBook();
}
```

我们在 aidl 文件中写代码时，Android Studio 不会帮我们自动导包，另外，尽管 Book.aidl 和 IBookManager.aidl 在同一个文件夹下，但是还是需要导入 Book。所以，啥也别说了，手写导包，首先把 Book.aidl 导入。然后定义两个方法。

好了，aidl 文件已经准备好了，接下来，Rebuild 一下项目。我们会发现在 `build/generated/source/aidl/debug/包名` 目录多了一个 IBookManager.java 文件。

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/aidl%E6%96%87%E4%BB%B6%E7%94%9F%E6%88%90Binder%E7%B1%BB.png"/>
</p>

我们把这个 IBookManager.java 文件拷贝到 aidl 文件夹中，和 Book.java 放在一起。然后把刚刚写的 Book.aidl 和 IBookManager.aidl 从系统生成的 aidl 文件夹中转移到我们自己新建的 aidl 文件夹中。最终目录结构如下：

<p align="center">
  <img src="https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/%E7%A7%BB%E5%8A%A8aidl%E6%96%87%E4%BB%B6.png"/>
</p>

之所以要移动 aidl 文件到我们自己创建的 aidl 目录，是因为 IBookManager.java 已经被我们移动到我们自己创建的 aidl 目录了，项目一编译，就会再根据 aidl 文件生成一个 IBookManager.java。这时就会报错。所以，为了让 aidl 文件不受编译影响，我们把 aidl 文件转移到我们自己创建的 aidl 文件夹中。