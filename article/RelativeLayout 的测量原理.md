在分析 RelativeLayout 的测量原理之前，我们先普及一些概念，因为 Relativelayout 的测量并不像 LinearLayout 那样直来直去，Relativelayout 的测量过程中涉及到了 rule（约束）。

那么，什么是 rule？

比如，我们在 xml 布局文件中设置了两个 TextView。在 text2 的属性中，我们看到有一个 `layout_toRightOf` 属性。我们在 xml 文件中设置的属性，在 RelativeLayout 的 Java 代码中也有对应的 int 类型常量字段：


```java
public class RelativeLayout extends ViewGroup {
    public static final int LEFT_OF                  = 0;
    public static final int RIGHT_OF                 = 1;
    ......
}
```

它们就叫做子元素（这里是 TextView）的 rule。

子元素的 rule 都保存在它对应的 LayoutParams 中的 mRule 中。


```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent">

    <TextView
        android:id="@+id/text1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:text="text1"/>

    <TextView
        android:id="@+id/text2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_toRightOf="@id/text1"
        android:text="text2"/>

</RelativeLayout>
```

以上面的布局文件为例，分析一下 getRelatedViewParams 方法：

这里我们分析的子元素是 text2，关联 View 是 text1。

```java
RelativeLayout # getRelatedViewParams

// 获取和子元素 text2 有 RIGHT_OF 布局关联的 View 的 LayoutParams
// 即 android:layout_toRightOf="@id/text1"，获取 id 为 text1 的 LayoutParams。
getRelatedViewParams(rules, RIGHT_OF)

private LayoutParams getRelatedViewParams(int[] rules, int relation) {
    // 获取和子元素 text2 有布局关联的那个 View text1
    View v = getRelatedView(rules, relation);
    if (v != null) {
        // 获取关联 View text1 的 LayoutParams
        ViewGroup.LayoutParams params = v.getLayoutParams();
        if (params instanceof LayoutParams) {
            return (LayoutParams) v.getLayoutParams();
        }
    }
    return null;
}

// 找出和子元素有布局关联的那个 View text1
private View getRelatedView(int[] rules, int relation) {
    // 因为子元素 text2 设置了 android:layout_toRightOf="@id/text1"（此时 relation 是 RIGHT_OF），
    // 那么 rules[relation] 会返回 text1 对应的 View 的 id。
    int id = rules[relation];
    if (id != 0) {
        // 通过 id 获取到 Node
        DependencyGraph.Node node = mGraph.mKeyNodes.get(id);
        if (node == null) return null;
        // 再通过 Node 获取到对应的 View
        View v = node.view;

        // Find the first non-GONE view up the chain
        // 如果 text1 对应的 View 是 GONE 状态，那就再看 text1 有没有
        // 设置 android:layout_toLeftOf="@id/xxx" 这个属性，
        // 如果设置了，就找出那个叫 xxx 的 View。
        while (v.getVisibility() == View.GONE) {
            rules = ((LayoutParams) v.getLayoutParams()).getRules(v.getLayoutDirection());
            node = mGraph.mKeyNodes.get((rules[relation]));
            if (node == null) return null;
            v = node.view;
        }

        return v;
    }

    return null;
}
```

再分析下 applyHorizontalSizeRules 方法：

```java
RelativeLayout # applyHorizontalSizeRules

private void applyHorizontalSizeRules(LayoutParams childParams, int myWidth, int[] rules) {
    // childParams 是 text2 的 LayoutParams
    // anchorParams 是 text1 的 LayoutParams
    RelativeLayout.LayoutParams anchorParams;

    // VALUE_NOT_SET indicates a "soft requirement" in that direction. For example:
    // left=10, right=VALUE_NOT_SET means the view must start at 10, but can go as far as it
    // wants to the right
    // left=VALUE_NOT_SET, right=10 means the view must end at 10, but can go as far as it
    // wants to the left
    // left=10, right=20 means the left and right ends are both fixed
    childParams.mLeft = VALUE_NOT_SET;
    childParams.mRight = VALUE_NOT_SET;

    // 确定 text2 的 mRight，也就是 text2 的右边缘的 x 坐标
    anchorParams = getRelatedViewParams(rules, LEFT_OF);
    if (anchorParams != null) {
        childParams.mRight = anchorParams.mLeft - (anchorParams.leftMargin +
                childParams.rightMargin);
    } else if (childParams.alignWithParent && rules[LEFT_OF] != 0) {
        if (myWidth >= 0) {
            childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
        }
    }

    // 以 RIGHT_OF 分析一下，
    // 因为在布局中我们给 text2 设置了 android:layout_toRightOf="@id/text1"，
    // 所以 getRelatedViewParams 会返回 text1 的 LayoutParams，
    anchorParams = getRelatedViewParams(rules, RIGHT_OF);
    if (anchorParams != null) {
        // （text1 的 LayoutParams 的 mRight，也就是 text1 的右边缘的 x 坐标） 
        // + （text1 的 LayoutParams 的 rightMargin，这里是 10dp） 
        // + (text2 的 LayoutParams 的 leftMargin，这里是 0dp)，
        // 最后得到 text2 的 mLeft，也就是 text2 的左边缘的 x 坐标。
        childParams.mLeft = anchorParams.mRight + (anchorParams.rightMargin +
                childParams.leftMargin);
    } else if (childParams.alignWithParent && rules[RIGHT_OF] != 0) {
        childParams.mLeft = mPaddingLeft + childParams.leftMargin;
    }

    // 再确定一次 text2 的 mLeft
    anchorParams = getRelatedViewParams(rules, ALIGN_LEFT);
    if (anchorParams != null) {
        childParams.mLeft = anchorParams.mLeft + childParams.leftMargin;
    } else if (childParams.alignWithParent && rules[ALIGN_LEFT] != 0) {
        childParams.mLeft = mPaddingLeft + childParams.leftMargin;
    }

    // 再确定一次 text2 的 mRight
    anchorParams = getRelatedViewParams(rules, ALIGN_RIGHT);
    if (anchorParams != null) {
        childParams.mRight = anchorParams.mRight - childParams.rightMargin;
    } else if (childParams.alignWithParent && rules[ALIGN_RIGHT] != 0) {
        if (myWidth >= 0) {
            childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
        }
    }

    // 再确定一次 text2 的 mLeft
    // 0 != rules[ALIGN_PARENT_LEFT]，表示没有设置 layout_alignParentLeft
    if (0 != rules[ALIGN_PARENT_LEFT]) {
        childParams.mLeft = mPaddingLeft + childParams.leftMargin;
    }

    // 再确定一次 text2 的 mRight
    // 0 != rules[ALIGN_PARENT_RIGHT]，表示没有设置 layout_alignParentRight
    if (0 != rules[ALIGN_PARENT_RIGHT]) {
        if (myWidth >= 0) {
            childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
        }
    }
}
```

总结一下，applyHorizontalSizeRules 方法会依次判断子元素和别的元素有没有 `LEFT_OF`、`RIGHT_OF`、`ALIGN_LEFT`、`ALIGN_RIGHT`、`ALIGN_PARENT_LEFT`、`ALIGN_PARENT_RIGHT` 这些关联，然后不停确定子元素的 mLeft 和 mRight，mLeft 和 mRight 一旦确定出来，子元素的宽就确定出来了。

宽度确定出来之后，会创建子元素的 MeasureSpec，然后调用子元素的 measure，这样测量过程就传递到子元素了。

```java
RelativeLayout # measureChildHorizontal

private void measureChildHorizontal(View child, LayoutParams params, int myWidth, int myHeight) {
    int childWidthMeasureSpec = getChildMeasureSpec(params.mLeft,
            params.mRight, params.width,
            params.leftMargin, params.rightMargin,
            mPaddingLeft, mPaddingRight,
            myWidth);
    int maxHeight = myHeight;
    if (mMeasureVerticalWithPaddingMargin) {
        maxHeight = Math.max(0, myHeight - mPaddingTop - mPaddingBottom -
                params.topMargin - params.bottomMargin);
    }
    int childHeightMeasureSpec;
    if (myHeight < 0 && !mAllowBrokenMeasureSpecs) {
        if (params.height >= 0) {
            childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(
                    params.height, MeasureSpec.EXACTLY);
        } else {
            // Negative values in a mySize/myWidth/myWidth value in RelativeLayout measurement
            // is code for, "we got an unspecified mode in the RelativeLayout's measurespec."
            // Carry it forward.
            childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
        }
    } else if (params.width == LayoutParams.MATCH_PARENT) {
        childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(maxHeight, MeasureSpec.EXACTLY);
    } else {
        childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(maxHeight, MeasureSpec.AT_MOST);
    }
    // 创建 childWidthMeasureSpec 和 childHeightMeasureSpec，传递到 child 的 measure 方法中。
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

宽确定出来，那高要怎么确定？这个通过 applyVerticalSizeRules 方法可以看到：

```java
RelativeLayout # applyVerticalSizeRules

private void applyVerticalSizeRules(LayoutParams childParams, int myHeight) {
    int[] rules = childParams.getRules();
    RelativeLayout.LayoutParams anchorParams;

    childParams.mTop = VALUE_NOT_SET;
    childParams.mBottom = VALUE_NOT_SET;

    anchorParams = getRelatedViewParams(rules, ABOVE);
    if (anchorParams != null) {
        childParams.mBottom = anchorParams.mTop - (anchorParams.topMargin +
                childParams.bottomMargin);
    } else if (childParams.alignWithParent && rules[ABOVE] != 0) {
        if (myHeight >= 0) {
            childParams.mBottom = myHeight - mPaddingBottom - childParams.bottomMargin;
        }
    }

    anchorParams = getRelatedViewParams(rules, BELOW);
    if (anchorParams != null) {
        childParams.mTop = anchorParams.mBottom + (anchorParams.bottomMargin +
                childParams.topMargin);
    } else if (childParams.alignWithParent && rules[BELOW] != 0) {
        childParams.mTop = mPaddingTop + childParams.topMargin;
    }

    anchorParams = getRelatedViewParams(rules, ALIGN_TOP);
    if (anchorParams != null) {
        childParams.mTop = anchorParams.mTop + childParams.topMargin;
    } else if (childParams.alignWithParent && rules[ALIGN_TOP] != 0) {
        childParams.mTop = mPaddingTop + childParams.topMargin;
    }

    anchorParams = getRelatedViewParams(rules, ALIGN_BOTTOM);
    if (anchorParams != null) {
        childParams.mBottom = anchorParams.mBottom - childParams.bottomMargin;
    } else if (childParams.alignWithParent && rules[ALIGN_BOTTOM] != 0) {
        if (myHeight >= 0) {
            childParams.mBottom = myHeight - mPaddingBottom - childParams.bottomMargin;
        }
    }

    if (0 != rules[ALIGN_PARENT_TOP]) {
        childParams.mTop = mPaddingTop + childParams.topMargin;
    }

    if (0 != rules[ALIGN_PARENT_BOTTOM]) {
        if (myHeight >= 0) {
            childParams.mBottom = myHeight - mPaddingBottom - childParams.bottomMargin;
        }
    }

    if (rules[ALIGN_BASELINE] != 0) {
        mHasBaselineAlignedChild = true;
    }
}
```

可以看到，applyVerticalSizeRules 方法的逻辑和 applyHorizontalSizeRules 方法的逻辑非常相似，applyVerticalSizeRules 方法会依次判断子元素和别的元素有没有 `ABOVE`、`BELOW`、`ALIGN_TOP`、`ALIGN_BOTTOM`、`ALIGN_PARENT_TOP`、`ALIGN_PARENT_BOTTOM` 这些关联，然后不停确定子元素的 mTop 和 mBottom，mTop 和 mBottom 一旦确定出来，子元素的高就确定出来了。

高确定出来之后，会调用 measureChild 方法，这样测量过程就传递到子元素了。

```java
RelativeLayout # measureChild

private void measureChild(View child, LayoutParams params, int myWidth, int myHeight) {
    int childWidthMeasureSpec = getChildMeasureSpec(params.mLeft,
            params.mRight, params.width,
            params.leftMargin, params.rightMargin,
            mPaddingLeft, mPaddingRight,
            myWidth);
    int childHeightMeasureSpec = getChildMeasureSpec(params.mTop,
            params.mBottom, params.height,
            params.topMargin, params.bottomMargin,
            mPaddingTop, mPaddingBottom,
            myHeight);
    // 创建 childWidthMeasureSpec 和 childHeightMeasureSpec，传递到 child 的 measure 方法中。
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

RelativeLayout 的 width 和 height：

通过前面的分析，我们知道了 RelativeLayout 测量子元素的流程。那么 RelativeLayout 自己的宽高是怎么确定下来的呢？

```java

@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    // RelativeLayout 初始宽度初始化为 -1
    int myWidth = -1;
    // RelativeLayout 初始高度初始化为 -1
    int myHeight = -1;

    // RelativeLayout 最终宽度初始化为 0
    int width = 0;
    // RelativeLayout 最终高度初始化为 0
    int height = 0;

    final int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    final int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    final int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    final int heightSize = MeasureSpec.getSize(heightMeasureSpec);

    if (widthMode != MeasureSpec.UNSPECIFIED) {
        myWidth = widthSize;
    }

    if (heightMode != MeasureSpec.UNSPECIFIED) {
        myHeight = heightSize;
    }

    if (widthMode == MeasureSpec.EXACTLY) {
        width = myWidth;
    }

    if (heightMode == MeasureSpec.EXACTLY) {
        height = myHeight;
    }

    // 到这里，RelativeLayout 的宽度和高度都是父容器的剩余大小

    // 如果 RelativeLayout 的宽度不是 EXACTLY，那么 isWrapContentWidth 为 true
    final boolean isWrapContentWidth = widthMode != MeasureSpec.EXACTLY;
    // 如果 RelativeLayout 的高度不是 EXACTLY，那么 isWrapContentHeight 为 true
    final boolean isWrapContentHeight = heightMode != MeasureSpec.EXACTLY;

    for (int i = 0; i < count; i++) {
        View child = views[i];
        if (child.getVisibility() != GONE) {
            LayoutParams params = (LayoutParams) child.getLayoutParams();
            
            ......

            // 如果 RelativeLayout 的宽度不是 EXACTLY
            if (isWrapContentWidth) {
                if (isLayoutRtl()) { // 如果布局方向是从右向左，具体流程这里不分析
                    if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                        width = Math.max(width, myWidth - params.mLeft);
                    } else {
                        width = Math.max(width, myWidth - params.mLeft - params.leftMargin);
                    }
                } else {
                    if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                        width = Math.max(width, params.mRight);
                    } else {
                        // RelativeLayout 会不停测量子元素的宽高，
                        // 然后拿（最右边的子元素的 mRight + rightMargin）作为自己的宽度值。
                        width = Math.max(width, params.mRight + params.rightMargin);
                    }
                }
            }

            // 如果 RelativeLayout 的高度不是 EXACTLY
            if (isWrapContentHeight) {
                if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                    height = Math.max(height, params.mBottom);
                } else {
                    // RelativeLayout 会不停测量子元素的宽高，
                    // 然后拿（最下边的子元素的 mBottom + bottomMargin）作为自己的高度值。
                    height = Math.max(height, params.mBottom + params.bottomMargin);
                }
            }

            ......
        }
    }

    // 最终，width 和 height 都确定了之后，就调用 setMeasuredDimension 方法，确定 RelativeLayout 的宽高。
    setMeasuredDimension(width, height);
}

```