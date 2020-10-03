我们知道，View 测量完了之后，View 的宽高就确定出来了。但是 View 宽高确定出来还没完，毕竟，View 最终是要显示在界面（手机屏幕）上的，既然要显示出来，那首先得知道要在哪个位置显示，也就是说要知道 View 的坐标。View 的坐标是在 layout 过程中确定的。

我们看下 View 的 layout 方法：

```java
public void layout(int l, int t, int r, int b) {
    ......

    int oldL = mLeft;
    int oldT = mTop;
    int oldB = mBottom;
    int oldR = mRight;

    // setFrame 方法会确定 View 的四个顶点的位置，
    // 四个顶点的位置一旦确定，View 的坐标也就确定了，
    // 那么 View 在父容器中的位置也就确定了。
    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
        // onLayout 用于父容器确定子元素的位置
        onLayout(changed, l, t, r, b);
        mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnLayoutChangeListeners != null) {
            ArrayList<OnLayoutChangeListener> listenersCopy =
                    (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
            int numListeners = listenersCopy.size();
            for (int i = 0; i < numListeners; ++i) {
                listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
            }
        }
    }

    ......
}
```

总结一下，在 `layout` 方法中，`setFrame` 方法会确定 View 的四个顶点的位置，进而确定 View 在父容器中的位置。然后调用 `onLayout` 方法，来确定子元素的位置。我们看一下 `onLayout` 方法：

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```

我们发现，`onLayout` 方法的实现是空的。嗯，什么情况？不是要确定子元素的位置吗。

我们仔细想想，好像 View 的 `onLayout` 方法还真不好实现，不同布局，它们子元素的摆放规则都不一样。所以 View 和 ViewGroup 都没有实现 `onLayout` 方法。所以要分析 `onLayout` 方法的实现，就要分析 LinearLayout 的 `onLayout` 实现或者 RelativeLayout 的 `onLayout` 实现。