View 的测量是由 `measure` 方法来完成的，在 `measure` 方法中，调用了 `onMeasure` 方法，所以我们来分析下 `onMeasure` 方法。

```java
View # onMeasure

protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

可以看到，`onMeasure` 里调用了 `setMeasuredDimension` 方法，`setMeasuredDimension` 方法有两个参数，根据参数的命名来看，`setMeasuredDimension` 的两个参数应该是 View 的宽度和高度。看下 `getDefaultSize` 方法：

```java
View # getDefaultSize

public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    // 一般我们自定义的 View 的测量模式不会是 UNSPECIFIED
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    // 当 View 的宽度是 wrap_content 时
    case MeasureSpec.AT_MOST:
    // 当 View 的宽度是具体尺寸（比如 100dp），或者 match_parent 时
    case MeasureSpec.EXACTLY:
        // 所以 View 的大小由 specSize 决定
        result = specSize;
        break;
    }
    return result;
}
``` 

我们可以看到，当 View 的测量模式是 `AT_MOST` 或者是 `EXACTLY` 时，View 的宽度是一样的，都是 specSize，也就是说，直接继承 View 的自定义控件，如果没有重写 `onMeasure` 方法并设置 `wrap_content` 时的大小，View 的宽高和 `match_parent` 时是一样的。

再看下 `getSuggestedMinimumWidth()` 方法：

```java
View # getSuggestedMinimumWidth

protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}
```

根据 getSuggestedMinimumWidth 的实现，可以看出，如果 mBackground 为空，也就是 View 没有设置背景，那么 View 的宽度是 mMinWidth，mMinWidth 对应于我们在 xml 文件中设置的 `android:minWidth` 属性。

```xml
<View
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:minWidth="10dp"/>
```

`android:minWidth` 属性如果没有设置，那么 `mMinWidth` 默认是 0。
如果 View 设置了背景，那么 View 的宽度是 `max(mMinWidth, mBackground.getMinimumWidth()`，

```java
Drawable # getMinimumWidth

public int getMinimumWidth() {
    final int intrinsicWidth = getIntrinsicWidth();
    return intrinsicWidth > 0 ? intrinsicWidth : 0;
}
```

`getMinimumWidth` 获取的是 Drawable 的原始宽度，如果 Drawable 没有原始宽度，getMinimumWidth 获取的就是 0。

虽然 `getSuggestedMinimumWidth()` 方法的逻辑不少，但是从之前的 `getDefaultSize` 方法可以看到。View 的宽高由 specSize 决定，和 `getSuggestedMinimumWidth()` 方法没什么关系。

最后总结一下，View 的测量过程由 `measure` 方法完成。一般我们自定义 View，需要重写 `onMeasure` 方法，如果是直接继承 View，那么还需要处理 `wrap_content` 时 View 的大小。

View 的大小由 View 的 MeasureSpec 的 `specSize` 决定，而 View 的 MeasureSpec 又由 View 的 LayoutParams 和父容器的 MeasureSpec 决定。