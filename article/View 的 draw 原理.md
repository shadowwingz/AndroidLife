在 View 的 `measure` 过程中，View 的宽高确定了，在 View 的 `layout` 过程中，View 的位置确定了，既然位置和宽高都确定了，那就只剩把 View 绘制到屏幕上了。

看一下 View 的 `draw` 方法：

```java
View # draw

public void draw(Canvas canvas) {
    final int privateFlags = mPrivateFlags;
    final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
            (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
    mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

    /*
     * Draw traversal performs several drawing steps which must be executed
     * in the appropriate order:
     *
     *      1. Draw the background
     *      2. If necessary, save the canvas' layers to prepare for fading
     *      3. Draw view's content
     *      4. Draw children
     *      5. If necessary, draw the fading edges and restore layers
     *      6. Draw decorations (scrollbars for instance)
     */

    // Step 1, draw the background, if needed
    int saveCount;

    if (!dirtyOpaque) {
        // 绘制背景
        drawBackground(canvas);
    }

    // skip step 2 & 5 if possible (common case)
    final int viewFlags = mViewFlags;
    boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
    boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
    if (!verticalEdges && !horizontalEdges) {
        // Step 3, draw the content
        // 绘制自己
        if (!dirtyOpaque) onDraw(canvas);

        // Step 4, draw the children
        // 绘制子元素
        dispatchDraw(canvas);

        // Step 6, draw decorations (scrollbars)
        // 绘制装饰
        onDrawScrollBars(canvas);

        if (mOverlay != null && !mOverlay.isEmpty()) {
            mOverlay.getOverlayView().dispatchDraw(canvas);
        }

        // we're done...
        return;
    }

    ......
}
```

总结一下，View 的绘制过程大概分为 4 步：

1. 绘制背景 `drawBackground(canvas);`
2. 绘制自己 `onDraw`
3. 绘制 children（`dispatchDraw`）
4. 绘制装饰（`onDrawScrollBars`）

重点说下 `dispatchDraw` 方法，这个方法是用来遍历所有子元素的 `draw` 方法。我们看下 View 的 `dispatchDraw` 方法的实现：

```java
View # dispatchDraw

protected void dispatchDraw(Canvas canvas) {
}
```

实现是空的，因为只有容器才会有子元素，那我们再看下 ViewGroup 的 `dispatchDraw` 方法：

```java
ViewGroup # dispatchDraw

protected void dispatchDraw(Canvas canvas) {
    ......
    for (int i = 0; i < childrenCount; i++) {
        int childIndex = customOrder ? getChildDrawingOrder(childrenCount, i) : i;
        final View child = (preorderedList == null)
                ? children[childIndex] : preorderedList.get(childIndex);
        if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {
            more |= drawChild(canvas, child, drawingTime);
        }
    }
    ......
}

ViewGroup # drawChild

protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
    return child.draw(canvas, this, drawingTime);
}
```

可以看到，在 ViewGroup 的 `dispatchDraw` 方法中，的确是遍历了所有子元素的 `draw` 方法。

最后总结一下：

如果不是容器，那么它的绘制流程是：

1. 绘制背景
2. 绘制自己
3. 绘制装饰

如果是容器，比如 LinearLayout，那么它的绘制流程是：

1. 绘制背景
2. 绘制自己
3. 绘制 children -> 每个子元素再调用自己的 draw 方法
4. 绘制装饰