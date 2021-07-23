看下 RelativeLayut 的 `onLayout` 方法：

```java
RelativeLayut # onLayout

@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    //  The layout has actually already been performed and the positions
    //  cached.  Apply the cached values to the children.
    final int count = getChildCount();

    for (int i = 0; i < count; i++) {
        View child = getChildAt(i);
        if (child.getVisibility() != GONE) {
            RelativeLayout.LayoutParams st =
                    (RelativeLayout.LayoutParams) child.getLayoutParams();
            child.layout(st.mLeft, st.mTop, st.mRight, st.mBottom);
        }
    }
}
```

我们发现，RelativeLayout 的 `layout` 过程比 LinearLayout 的更简单，只是遍历了每个子元素，然后调用子元素的 `layout` 方法。

我们回忆一下 LinearLayout 的 `layout` 过程，看下 LinearLayout 和 RelativeLayout `layout` 的过程有什么区别。

1. `childTop` 变量

在 LinearLayout 中，有一个 `childTop` 变量，每个子元素的 `layout` 都依赖于变量来确定自己的位置，后面的子元素会被放置在靠下的位置。

在 RelativeLayout 中，并没有 `childTop` 变量，那子元素的位置怎么确定呢？答案是在 RelativeLayout 的 `measure` 过程中，子元素的四个顶点的位置就已经确定了。

```java
RelativeLayout  # applyHorizontalSizeRules

private void applyHorizontalSizeRules(LayoutParams childParams, int myWidth, int[] rules) {
    childParams.mLeft;
    childParams.mRight;
}

RelativeLayout  # applyVerticalSizeRules

private void applyVerticalSizeRules(LayoutParams childParams, int myHeight) {
    childParams.mTop;
    childParams.mBottom;
}
```

而在 LinearLayout 的 `measure` 过程中，只是确定了子元素的宽高：

```java
LinearLayout # measureChildBeforeLayout

void measureChildBeforeLayout(View child, int childIndex,
            int widthMeasureSpec, int totalWidth, int heightMeasureSpec,
            int totalHeight) {
    measureChildWithMargins(child, widthMeasureSpec, totalWidth,
            heightMeasureSpec, totalHeight);
}

View # measureChildWithMargins

protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
    final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

    // 把宽度封装到了 childWidthMeasureSpec 中
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    // 把高度封装到了 childHeightMeasureSpec 中
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

但是具体位置还没确定，所以在 LinearLayout 的 `layout` 过程中还需要 `childTop` 来辅助确定位置。

2. 方向

LinearLayout 有两个方向，竖直和水平，而在 `layout` 的过程中，也是按照两个方向来 `layout` 的。这个从源码中可以看出来：

```java
LinearLayout # onLayout

@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        layoutVertical(l, t, r, b);
    } else {
        layoutHorizontal(l, t, r, b);
    }
}
```

而 RelativeLayout 由于不需要设置方向，所以也不需要按照两个方向来 `layout`。