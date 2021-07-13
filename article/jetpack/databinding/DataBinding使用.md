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