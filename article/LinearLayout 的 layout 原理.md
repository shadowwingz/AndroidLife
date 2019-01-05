在 View 的 `layout` 原理 中，我们知道了，View 的 `layout` 方法会确定自己在父容器中的位置，而要确定子元素在 View 自己中的位置，是由 `onLayout` 方法完成的。由于不同布局的实现不一样，所以 ViewGroup 也没有实现 `onLayout` 方法，而是交由具体的布局去实现。

```java
ViewGroup # onLayout

protected abstract void onLayout(boolean changed,
            int l, int t, int r, int b);
```

下面我们分析下 LinearLayout 的 `layout` 原理，首先看下 LinearLayout 的 `onLayout` 方法：

```java
LinearLayout # onLayout

@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    if (mOrientation == VERTICAL) {
        // 如果 LinearLayout 是竖直方向，就调用这个方法
        layoutVertical(l, t, r, b);
    } else {
        // 如果 LinearLayout 是水平方向，就调用这个方法
        layoutHorizontal(l, t, r, b);
    }
}
```

以竖直方向为例，我们看下 layoutVertical 方法：

```java
LinearLayout # layoutVertical

void layoutVertical(int left, int top, int right, int bottom) {
    ......

    // 遍历子元素
    for (int i = 0; i < count; i++) {
        final View child = getVirtualChildAt(i);
        if (child == null) {
            childTop += measureNullChild(i);
        } else if (child.getVisibility() != GONE) {
            final int childWidth = child.getMeasuredWidth();
            final int childHeight = child.getMeasuredHeight();
            
            final LinearLayout.LayoutParams lp =
                    (LinearLayout.LayoutParams) child.getLayoutParams();
            
            ......

            if (hasDividerBeforeChildAt(i)) {
                // childTop 逐渐增大
                childTop += mDividerHeight;
                }

                childTop += lp.topMargin;
                // 调用子元素的 layout 方法
                setChildFrame(child, childLeft, childTop + getLocationOffset(child),
                        childWidth, childHeight);
                childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);

                i += getChildrenSkipCount(child, i);
            }
        }
    }
```

总结一下，LinearLayout 会遍历子元素并调用 `setChildFrame` 方法，`setChildFrame` 是用来确定子元素的位置。在遍历的过程中，`childTop` 会不断增大，后面的子元素会被放置在靠下的位置。看下 `setChildFrame` 方法：

```java
LinearLayout # setChildFrame

private void setChildFrame(View child, int left, int top, int width, int height) {        
    // left, left + width 分别是 View 左右边缘的 x 坐标
    // top, top + height 分别是 View 上下边缘的 y 坐标
    child.layout(left, top, left + width, top + height);
}
```

可以看到，`setChildFrame` 方法中只是调用了子元素的 `layout` 方法。这样，子元素的位置就确定了。

最后总结一下，LinearLayout 的 `layout` 过程中，会遍历子元素，并把子元素从上下到依次排列。