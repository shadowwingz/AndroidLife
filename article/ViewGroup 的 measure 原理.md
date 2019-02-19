首先，ViewGroup （父容器）会测量自己的大小，要测量自己的大小，就得先测量所有子元素的大小，毕竟，父容器的大小可以说就是所有子元素的大小。

测量子元素的大小，父容器使用的是 `measureChildren` 方法，在这个方法里，父容器会遍历所有子 View，然后依次调用每个子元素的 measure 方法：

```java
ViewGroup # measureChildren

protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    // 遍历子元素
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        // 如果子元素的状态不是 GONE
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
            // 就测量子元素
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}
```

再看下 `measureChild` 方法：

```java
ViewGroup # measureChild

protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
    // 获取子元素的 LayoutParams
    final LayoutParams lp = child.getLayoutParams();

    // 使用父容器的 MeasureSpec 
    // 和子元素的 LayoutParams 来创建子元素的 MeasureSpec 
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);

    // 把子元素的 MeasureSpec 传递给子元素的 measure 方法
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

总结一下，父容器会根据自己的 MeasureSpec 和子元素的 LayoutParams 来创建子元素的 MeasureSpec，然后子元素就拿着这个 MeasureSpec 去测量自己的尺寸去了。

父容器的 MeasureSpec 是从哪来的呢？是根据父容器的父容器的 MeasureSpec 和父容器自己的 LayoutParams 创建的。

子元素的 LayoutParams 是从哪来的呢？是根据布局文件里写的尺寸参数，比如 `match_parent` ，或者具体的尺寸创建的。

总之，子元素有了 MeasureSpec，接着，子元素就去测量自己的尺寸去了，具体的流程已经在 [View 的 draw 原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/View%20%E7%9A%84%20draw%20%E5%8E%9F%E7%90%86.md) 中分析了，这里就不再多说了。

子元素测量好了，接下来就该测量父容器自己了。但是我们在 ViewGroup 源码中可以发现，ViewGroup 并没有重写 `onMeasure` 方法。这是为什么呢？因为根本就没法重写，没办法，不同的 ViewGroup 子类有自己独特的布局特性，回忆一下我们使用的 LinearLayout 和 RelativeLayout，它们使用起来差异就很大，所以测量细节肯定是不同的。具体测量细节请看

LinearLayout 的测量原理

RelativeLayout 的测量原理