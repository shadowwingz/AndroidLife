在自定义 View 的过程中，都少不了 View 的测量，而在 View 的测量过程中，又经常要和 MeasureSpec 打交道。所以，要学好自定义 View，首先要弄懂 MeasureSpec 的原理。

MeasureSpec 代表一个 32 位 int 值，高 2 位代表 SpecMode，低 30 位代表 SpecMode，SpecMode 是指测量模式，SpecSize 是指在某种测量模式下的规格大小。

SpecMode 有三类，含义如下：

### UNSPECIFIED ###
 
父容器不对 View 有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量的状态。

### EXACTLY ###

父容器已经检测出 View 所需要的精确大小，这个时候 View 的最终大小就是 SpecSize 所指定的值。它对应于 LayoutParams 中的 `match_parent` 和具体的数值这两种模式。

### AT_MOST ###

父容器指定了一个大小即 SpecSize，View 的大小不能大于这个值，具体是什么值要看不同 View 的具体实现（如果不做特殊处理，View 的大小一般就是这个值）。它对应于 LayoutParams 中的 `wrap_content`。

接下来，我们看一下 View 的测量过程。

对于普通 View 来说，View 的 measure 过程由 ViewGroup 传递而来，先看一下 ViewGroup 的 `measureChildWithMargins` 方法：

```java
ViewGroup # measureChildWithMargins

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

上述方法会对子元素进行 measure，在调用子元素的 measure 方法之前会先通过 `getChildMeasureSpec` 方法来得到子元素的 MeasureSpec，从代码来看，很显然，子元素的 MeasureSpec 的创建与父容器的 MeasureSpec 和子元素本身的 LayoutParams 有关，再看下 `getChildMeasureSpec` 方法：

```java
ViewGroup # getChildMeasureSpec

// spec 是父容器的 MeasureSpec
// padding 是父容器中已占有的空间大小
// childDimension 是子元素想要的尺寸
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
    // 取出父容器的测量模式
    int specMode = MeasureSpec.getMode(spec);
    // 取出父容器的测量大小
    int specSize = MeasureSpec.getSize(spec);

    // 父容器的尺寸，也是子元素可用的大小
    int size = Math.max(0, specSize - padding);

    // 子元素的测量大小
    int resultSize = 0;
    // 子元素的测量模式
    int resultMode = 0;

    switch (specMode) {
    // Parent has imposed an exact size on us
    // 如果父容器的测量模式是 EXACTLY
    case MeasureSpec.EXACTLY:
        // 如果子元素的尺寸大于 0，比如 100 dp
        if (childDimension >= 0) {
            // 那么子元素的测量大小就是 100 dp
            resultSize = childDimension;
            // 子元素的测量模式就是 EXACTLY
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // 如果子元素的尺寸是 match_parent
            // Child wants to be our size. So be it.
            // 那么子元素的测量大小就是父容器的尺寸
            resultSize = size;
            // 子元素的测量模式就是 EXACTLY
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            // 如果子元素的尺寸是 wrap_content
            // 那么子元素的测量大小就是父容器的尺寸
            resultSize = size;
            // 子元素的测量模式就是 AT_MOST
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent has imposed a maximum size on us
    // 如果父容器的测量模式是 AT_MOST
    case MeasureSpec.AT_MOST:
        // 如果子元素的尺寸大于 0，比如 100 dp
        if (childDimension >= 0) {
            // Child wants a specific size... so be it
            // 那么子元素的测量大小就是 100 dp
            resultSize = childDimension;
            // 子元素的测量模式就是 EXACTLY
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size, but our size is not fixed.
            // Constrain child to not be bigger than us.
            // 如果子元素的尺寸是 match_parent
            // 那么子元素的测量大小就是父容器的尺寸
            resultSize = size;
            // 子元素的测量模式就是 AT_MOST
            resultMode = MeasureSpec.AT_MOST;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size. It can't be
            // bigger than us.
            // 如果子元素的尺寸是 wrap_content
            // 那么子元素的测量大小就是父容器的尺寸
            resultSize = size;
            // 子元素的测量模式就是 AT_MOST
            resultMode = MeasureSpec.AT_MOST;
        }
        break;

    // Parent asked to see how big we want to be
    // UNSPECIFIED 这个模式主要用于系统内部多次 measure 的情形，一般来说，我们不需要关注此模式
    case MeasureSpec.UNSPECIFIED:
        if (childDimension >= 0) {
            // Child wants a specific size... let him have it
            resultSize = childDimension;
            resultMode = MeasureSpec.EXACTLY;
        } else if (childDimension == LayoutParams.MATCH_PARENT) {
            // Child wants to be our size... find out how big it should
            // be
            resultSize = 0;
            resultMode = MeasureSpec.UNSPECIFIED;
        } else if (childDimension == LayoutParams.WRAP_CONTENT) {
            // Child wants to determine its own size.... find out how
            // big it should be
            resultSize = 0;
            resultMode = MeasureSpec.UNSPECIFIED;
        }
        break;
    }
    return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

上面的代码可能有些不好理解，下面我们用 xml 布局文件来进一步说明：

#### 父容器的测量模式是 EXACTLY： ####

```xml
<!-- 父容器的测量模式是 EXACTLY -->
<RelativeLayout
    android:layout_width="200dp"
    android:layout_height="200dp">

    <!-- 子元素的尺寸大于 0，那么子元素的实际尺寸是 100 dp，
    此时子元素的测量模式也是 EXACTLY -->
    <View
        android:layout_width="100dp"
        android:layout_height="100dp"/>

    <!-- 子元素的尺寸是 match_parent，那么子元素的实际尺寸是父容器的尺寸，
    此时子元素的测量模式是 AT_MOST -->
    <View
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

    <!-- 子元素的尺寸是 wrap_content，那么子元素的实际尺寸是父容器的尺寸，
    此时子元素的测量模式是 AT_MOST -->
    <View
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</RelativeLayout>
```

#### 父容器的测量模式是 AT_MOST： ####

```xml
<!-- 父容器的测量模式是 AT_MOST -->
<RelativeLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content">

    <!-- 子元素的尺寸大于 0，那么子元素的实际尺寸是 100 dp，
   此时子元素的测量模式也是 EXACTLY -->
    <View
        android:layout_width="100dp"
        android:layout_height="100dp"/>

    <!-- 子元素的尺寸是 match_parent，那么子元素的实际尺寸是父容器的尺寸，
    此时子元素的测量模式是 AT_MOST -->
    <View
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

    <!-- 子元素的尺寸是 wrap_content，那么子元素的实际尺寸是父容器的尺寸，
    此时子元素的测量模式是 AT_MOST -->
    <View
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
</RelativeLayout>
```

总结一下，

当 View 采用固定宽/高（比如 100dp）时，
不管父容器的 MeasureSpec 是什么，
View 的 MeasureSpec 都是 EXACTLY，并且其大小遵循 LayoutParams 中的大小，也就是 100dp。

当 View 的宽高是 `match_parent` 时，
如果父容器的模式是 EXACTLY（`match_parent` 或 100 dp），那么 View 也是 EXACTLY 并且其大小是父容器的剩余空间；
如果父容器是 AT_MOST（`wrap_content`），那么 View 也是最大模式并且其大小不会超过父容器的剩余空间（一般情况下就是父容器的大小）。

当 View 的宽高是 `wrap_content` 时，
不管父容器的模式是 EXACTLY 还是 AT_MOST，
View 的模式总是 AT_MOST 并且大小不能超过父容器的剩余空间（一般情况下就是父容器的大小）。

到这里，可能有些同学有疑问了，为什么我们平常给 TextView 设置 `wrap_content` 时，控件都是紧紧把文字包裹住，而不是像我们在上面说的，填充满整个父容器。那是因为在 TextView 的 `onMeasure` 方法中，TextView 对 `wrap_content` 这种情况做了特殊处理，从而使 TextView 紧紧把文字包裹住。