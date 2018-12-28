在 [ViewGroup 的测量原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/ViewGroup%20%E7%9A%84%E6%B5%8B%E9%87%8F%E5%8E%9F%E7%90%86.md) 中我们提到，ViewGroup 是没有重写 `onMeasure` 方法的，因为不用的 ViewGroup 子类的测量方式不同，所以没办法直接在 ViewGroup 中重写，只能在 LinearLayout 或者 ReletiveLayout 这种具体子类中重写。

那么看下 LinearLayout 是怎么测量的，首先看下 `onMeasure` 方法：

```java
LinearLayout # onMeasure

@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    if (mOrientation == VERTICAL) {
        measureVertical(widthMeasureSpec, heightMeasureSpec);
    } else {
        measureHorizontal(widthMeasureSpec, heightMeasureSpec);
    }
}
```

可以看到，LinearLayout 首先会判断自己的方向，

如果是竖直方向，就调用 `measureVertical` 方法，

如果是水平方向，就调用 `measureHorizontal` 方法。

我们先看 `measureVertical` 方法：

```java
LinearLayout # measureVertical

void measureVertical(int widthMeasureSpec, int heightMeasureSpec) {
    ......

    // See how tall everyone is. Also remember max width.
    // 遍历子元素
    for (int i = 0; i < count; ++i) {
        // 获取竖直方向的子元素
        final View child = getVirtualChildAt(i);

       ......

        // Determine how big this child would like to be. If this or
        // previous children have given a weight, then we allow it to
        // use all available space (and we will shrink things later
        // if needed).
        // 测量子元素的宽高
        measureChildBeforeLayout(
               child, i, widthMeasureSpec, 0, heightMeasureSpec,
               totalWeight == 0 ? mTotalLength : 0);

        if (oldHeight != Integer.MIN_VALUE) {
           lp.height = oldHeight;
        }

        // 获取子元素的高度
        final int childHeight = child.getMeasuredHeight();
        final int totalLength = mTotalLength;
        // 把子元素的高度加到 mTotalLength 里
        // mTotalLength 就是 LinearLayout 的总高度
        mTotalLength = Math.max(totalLength, totalLength + childHeight + lp.topMargin +
               lp.bottomMargin + getNextLocationOffset(child));

        ......
    }
}

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

    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                    + widthUsed, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                    + heightUsed, lp.height);

    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

measureVertical 的逻辑比较复杂，我们先把流程简单化一点。

假设 LinearLayout 里面的子元素的高度都是有数值的，比如 `100dp`，那么 LinearLayout 测量的时候，就会遍历竖直方向上的子元素，然后调用子元素的 `measure` 方法，这样各个子元素就开始了 `measure` 过程，具体请看 [View 的测量原理](https://github.com/shadowwingz/AndroidLife/blob/master/article/View%20%E7%9A%84%E6%B5%8B%E9%87%8F%E5%8E%9F%E7%90%86.md)， 子元素 `measure` 过程完成后，子元素的宽度和高度就确定了。然后把子元素的高度和子元素在竖直方向的 `margin` 加到 `mTotalLength` 里。

```java
LinearLayout # measureVertical

// Add in our padding
// 把 LinearLayout 竖直方向的 padding 加到 mTotalLength 里
mTotalLength += mPaddingTop + mPaddingBottom;

// 确定 LinearLayout 的高度
int heightSize = mTotalLength;

// Check against our minimum height
heightSize = Math.max(heightSize, getSuggestedMinimumHeight());

// Reconcile our calculated size with the heightMeasureSpec
int heightSizeAndState = resolveSizeAndState(heightSize, heightMeasureSpec, 0);
heightSize = heightSizeAndState & MEASURED_SIZE_MASK;
```

总结一下，如果是竖直方向的 LinearLayout，它的高度就是所有子元素的高度 + 所有子元素在竖直方向上的 margin + LinearLayout 在竖直方向的 padding。

我们再看下 LinearLayout 宽度的测量：

```java
LinearLayout # measureVertical

for (int i = 0; i < count; ++i) {
    final View child = getVirtualChildAt(i);

    ......

    final int measuredWidth = child.getMeasuredWidth() + margin;
    maxWidth = Math.max(maxWidth, measuredWidth);
}
```

可以发现，LinearLayout 会遍历它的每一个子元素，看哪个子元素的宽度最大，就拿那个子元素的宽度作为 LinearLayout 自己的宽度。

分析到这里，我们发现 LinearLayout 的宽度和高度都有了，但是还没完，我们先看下最终决定 LinearLayout 宽高的代码：

```java
LinearLayout # measureVertical

mTotalLength += mPaddingTop + mPaddingBottom;
int heightSize = mTotalLength;
heightSize = Math.max(heightSize, getSuggestedMinimumHeight());
int heightSizeAndState = resolveSizeAndState(heightSize, heightMeasureSpec, 0);

setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                heightSizeAndState);
```

我们可以看到，`mTotalLength` 赋值给了 `heightSize`，然后，`heightSize` 和 `maxWidth` 都被传入了 resolveSizeAndState 方法，`resolveSizeAndState` 方法的返回值被作为了最终宽高传入了 `setMeasuredDimension` 方法。我们看下 `resolveSizeAndState` 方法：

```java
LinearLayout # resolveSizeAndState

// size 是 LinearLayout 测量之后得到的大小，也是 LinearLayout 想要的大小
// measureSpec 是 LinearLayout 自己的 LayoutParams 和父容器的 MeasureSpec 共同作用生成的 MeasureSpec
public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
    int result = size;
    // LinearLayout 的测量模式
    int specMode = MeasureSpec.getMode(measureSpec);
    // LinearLayout 的测量大小，也是 LinearLayout 父容器的剩余空间
    int specSize =  MeasureSpec.getSize(measureSpec);
    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    // 如果 LinearLayout 的测量模式是 AT_MOST，即布局使用的是 wrap_content
    case MeasureSpec.AT_MOST:
        // 如果 LinearLayout 想要的大小比父容器的剩余空间大
        if (specSize < size) {
            // 那么 LinearLayout 的大小就是父容器的剩余空间
            result = specSize | MEASURED_STATE_TOO_SMALL;
        } else {
            // 否则 LinearLayout 的大小就是它想要的大小
            result = size;
        }
        break;
    // 如果 LinearLayout 的测量模式是 EXACTLY，
    // 即布局使用的是 match_parent 或者具体数值    
    case MeasureSpec.EXACTLY:
        // LinearLayout 的大小就是它想要的大小
        result = specSize;
        break;
    }
    return result | (childMeasuredState&MEASURED_STATE_MASK);
}
```

总结一下，

如果 LinearLayout 的测量模式是 EXACTLY，那 LinearLayout 想要多大就给它多大，

如果 LinearLayout 的测量模式是 AT_MOST，那此时要看父容器的大小，要是 LinearLayout 想要的大小比父容器的剩余空间小，那就要多大给多大，要是 LinearLayout 想要的大小比父容器的剩余空间还大，那父容器也无能为力，只能把剩余空间全部给 LinearLayout，再要多的也没有。