看下 LinearLayout 的 `onDraw` 方法：

```java
LinearLayout # onDraw

@Override
protected void onDraw(Canvas canvas) {
    if (mDivider == null) {
        return;
    }

    if (mOrientation == VERTICAL) {
        drawDividersVertical(canvas);
    } else {
        drawDividersHorizontal(canvas);
    }
}
```

首先，LinearLayout 会判断 `mDivider` （分割线）是否为空，`mDivider` 属性一般在 xml 布局文件中设置：

```xml
<LinearLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:divider="@android:color/white"/>
```

如果设置了 `android:divider` 属性，那么 LinearLayout 就会根据自己的布局方向去遍历子元素，然后绘制分割线。

如果没有设置 `android:divider` 属性，那么 `mDivider` 就会为空，然后就完了。draw 过程结束。

有的童鞋可能会一脸懵逼，

- 卧槽，这就完了？
- 是的，这就完了。
- 那 LinearLayout 和它里面的子元素到底是怎么绘制出来的？
- 在 View 的 `draw` 方法中绘制出来的。

首先，LinearLayout 是继承自 ViewGroup 的，然后，ViewGroup 又是继承自 LinearLayout 的，So，LinearLayout 的 `draw` 流程，是从 View 的 `draw` 方法开始的。

我们回忆下 [View 的 draw 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/View%20%E7%9A%84%20draw%20%E5%8E%9F%E7%90%86.md)，如果是容器，它的绘制流程是：

1. 绘制背景
2. 绘制自己
3. 绘制 children -> 每个子元素再调用自己的 `draw` 方法
4. 绘制装饰

上面四个流程，已经把 LinearLayout 给绘制出来了。那有的童鞋可能会问，既然 View 的 `draw` 方法都能把 LinearLayout 绘制出来，那 LinearLayout 还干嘛要重写 `onDraw` 方法。

因为每个布局都有可以复用的绘制逻辑，也就是 View 的 `draw` 方法的逻辑。而每个布局又都有自己的一些布局特点，比如 LinearLayout，它就有个分割线 `mDivider`，所以这个 `mDivider` 的绘制过程就被放到 LinearLayout 的 `onDraw` 方法中去实现了。

那如果有的布局没有什么特点呢？那就不重写 `onDraw` 方法。不信你看 RelativeLayout，它就没有 `onDraw` 方法。