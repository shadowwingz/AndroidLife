### DataBinding 预览默认值

我们在布局的 xml 文件中，如果使用了 DataBinding，一般是这个样子

```xml
<TextView
    android:id="@+id/tv_result"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="50dp"
    android:text="@{data.date}"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

这样写有个小问题，就是在 Android Studio 的预览窗口里看不到 TextView 的内容。这个也可以理解，毕竟现在我们的 TextView 绑定的数据是 `data.date`，而 `data.date` 的值并不确定，需要运行起来才知道。

不过，话是这么说，如果我非要在预览窗口里看到这个 TextView 的内容呢？也是可以的，我们给这个 TextView 指定一个 `default` 值就可以了。

```xml
<TextView
    android:id="@+id/tv_result"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="50dp"
    android:text="@{data.date, default=默认值}"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
```

我们给 text 设置了 `@{data.date, default=默认值}`，其中 `default` 的值我们可以在预览窗口中看到。

或者还有一种简单的方法： `tools:xxx` 属性，比如在这里我们可以用 `tools:text`，给 text 设置一个默认文本，这个文本仅在预览框中生效。

```xml
<TextView
    android:id="@+id/tv_result"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginTop="50dp"
    android:text="@{data.date"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toTopOf="parent"
    tools:text="默认值" />
```

### DataBinding 的基本使用：

如果是在 Activity 中使用 DataBinding，我们一般是用布局文件或者 inflater 来初始化一个 binding：

```kotlin
// 1. 使用布局文件初始化 binding
val binding: ActivityDataBindingBinding = DataBindingUtil.setContentView(this, R.layout.activity_data_binding)
// 2. 使用 layoutInflater 初始化 binding
val binding2: ActivityDataBindingBinding = ActivityDataBindingBinding.inflate(layoutInflater)
```

这个 binding 是什么？如果我们直接点进去 binding，会跳转到对应的布局文件中，实际上，每个布局文件在 `app/build/generated/data_binding_base_class_source_out/debug/out/包路径/databinding` 下都会生成对应的 Binding 文件。在这个 Binding 文件中，我们可以看到我们在布局文件中定义的控件：

```java
@NonNull
public final TextView tvResult;
```

那也就是说，一旦我们有了这个 binding，如果我们要获取一个控件，不用再 `findViewById` 了，直接通过 binding 去拿这个控件就行了。

比如我们给 TextView 设置一个 text，我们就可以这样：

```kotlin
binding.tvResult.text = "test"
```

这是给一个控件赋值，那如果我们有一个实体类，这个实体类里有多个字段，需要分别赋值给界面上的不同控件，那我们要怎么做呢？我们的第一反应可能是这样：

```kotlin
// user 是实体类对象
binding.tvName.text = user.name
binding.tvAge.text = user.age
```

这样虽说没啥问题，但是效率有点低，为了提升效率，我们可以直接把 user 对象赋值给 binding，

第一步：

在布局文件中，引入我们的实体类

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:binding="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="user"
            type="包名.jetpack.databinding.User" />

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".jetpack.databinding.DataBindingActivity">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:orientation="vertical"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@id/tv_result">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{user.firstName}" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{user.lastName}" />
        </LinearLayout>

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

User 类：

```kotlin
data class User(val firstName: String, val lastName: String)
```

我们在布局中的 `data` 标签中引入了我们的实体类

```xml
<variable
    name="user"
    type="包名.jetpack.databinding.User" />
```

name 是实体类的别名，这个别名有 2 个用途，一个用途是在布局文件内通过这个别名来引用字段，比如我们把 user 对象的 firstName 和 TextView `绑定`起来，就可以这样：

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.firstName}" />
```

另一个用途是，在 java 文件中通过这个别名来赋值：

```kotlin
binding.user = User("Test", "User")
```

看似是 2 个用途，实际上它们在使用中也是密不可分的。先绑定，再赋值。

我们把一个实体类对象赋值给 `binding.user`，这个实体类对象就会把它的每个字段再赋值给它所绑定的每一个控件。在这里例子中就是，我们把一个 User 对象赋值给 `binding.user`，这个 user 对象的 firstName 值和 lastName 值就分别设置给了对应的 TextView。

### RecyclerView 中使用 DataBinding

和 Activity 中类似，RecyclerView 中使用 Binding 分为 2 步：

1. 创建列表 item 布局文件对应的 binding
2. 给 binding 内部的数据赋值

代码大概类似于这个样子：

```kotlin
class DemoListAdapter : ListAdapter<ListBean, RecyclerView.ViewHolder>(UserDiffCallback()) {

  // 这里不用再 findViewById 实例化 View
  class ViewHolder(view: View) : RecyclerView.ViewHolder(view)

  // 使用 DiffUtil，提高刷新效率
  class UserDiffCallback : DiffUtil.ItemCallback<ListBean>() {

    override fun areItemsTheSame(oldItem: ListBean, newItem: ListBean): Boolean {
      return oldItem.title == newItem.title
    }

    override fun areContentsTheSame(oldItem: ListBean, newItem: ListBean): Boolean {
      return oldItem == newItem
    }
  }

  override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
    // 使用 DataBindingUtil.inflate 获取布局文件对应的 binding
    val binding: DemoListBinding = DataBindingUtil.inflate(
      LayoutInflater.from(parent.context), R.layout.demo_list, parent, false
    )
    return ViewHolder(binding.root)
    // val rootView = LayoutInflater.from(parent.context).inflate(R.layout.demo_list, parent, false)
    // return ViewHolder(rootView)
  }

  override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
    // 先通过 View 获取到对应的 binding
    val binding: DemoListBinding? = DataBindingUtil.getBinding(holder.itemView)
    // 再通过 getItem 拿到对应的 item 数据，赋值给 binding.data，完成刷新
    binding?.data = getItem(position)
  }
}
```

核心就是 `onCreateViewHolder` 和 `onBindViewHolder` ，在 `onCreateViewHolder` 中我们使用 DataBindingUtil.inflate 获取了布局文件对应的 binding，然后在 `onBindViewHolder` 中我们通过 holder 拿到 itemView 对应的 binding，最后给 binding.data 赋值。这里的 binding.data 就是 item 布局文件中引入的 data：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

        <variable
            name="data"
            type="com.shadowwingz.androidpractice.jetpack.databinding.list.ListBean" />
    </data>

    <!-- 省略了下面的布局 -->

</layout>
```