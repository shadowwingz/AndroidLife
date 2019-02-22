在 [View 的 measure 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/View%20%E7%9A%84%20measure%20%E5%8E%9F%E7%90%86.md) 中，我们提到了：

> View 的大小由 View 的 MeasureSpec 的 specSize 决定，而 View 的 MeasureSpec 又由 View 的 LayoutParams 和父容器的 MeasureSpec 决定。

MeasureSpec 在 [MeasureSpec 源码解析](https://github.com/shadowwingz/AndroidLife/blob/master/article/MeasureSpec%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md) 中已经分析过了，这里就不再多说了。我们这里分析下 LayoutParams。

#### LayoutParams 是什么？

其实我们一直都在和 LayoutParams 打交道，回忆一下我们写布局文件的时候，是不是经常要写 `layout_width` 和 `layout_height`。比如：

```java
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_marginLeft="10dp"
    android:text="测试"
    android:textColor="@color/colorAccent"
    android:textSize="16sp" />
```

话说，刚学安卓的时候，我就在疑惑，为什么有的属性，前面不带 `layout` 前缀，比如 `text`、`textColor `、`textSize `，而有的属性，前面要带 `layout` 前缀，比如 `layout_width `，`layout_height `、`layout_marginLeft `。

用通俗的话讲，不带 `layout` 前缀的属性都是只和当前 View 相关的。

- 比如 `text `，View 要显示什么文字，就显示文字
- 比如 `textColor `，View 要显示什么颜色，就显示颜色
- 比如 `textSize `，View 的字体多大，就是多大

而带 `layout ` 前缀的属性呢？它们不但和当前 View 相关，还和当前 View 的父布局相关。

- 比如 `layout_width`，当这个属性是 `match_parent` 时，View 的宽度是父布局的剩余宽度，也就是说，View 想要多大的宽度，得看父布局还有多大的宽度。
- 比如 `layout_marginLeft`，当这个属性是 `0dp` 时，如果 View 的父布局紧贴屏幕左边缘，那么 View 也会紧贴屏幕左边缘，但是如果 View 的父布局紧贴屏幕右边缘,那 View 离屏幕左边缘就远了去了。

讲到这里，我们就可以解开谜底了，LayoutParams 到底是什么？LayoutParams 就是带 `layout` 前缀属性的集合，是 View 用来告诉它的父控件如果放置自己。

### LayoutParams 的使用

前面说到的 `layout_width `、`layout_height ` 是在 xml 布局文件中使用的，可以一边写一边看效果。但是如果想动态改变 View 宽高，那还是得用 LayoutParams 来实现。我们来实现一下：