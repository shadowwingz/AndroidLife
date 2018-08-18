png

<img src="https://github.com/shadowwingz/AndroidLife/blob/master/art/AlertDialog%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.png"/>

jpg

<img src="https://github.com/shadowwingz/AndroidLife/blob/master/art/AlertDialog%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.jpg"/>

svg

<img src="https://github.com/shadowwingz/AndroidLife/blob/master/art/AlertDialog%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.svg"/>

AlertDialog 源码使用的是 Builder 模式，通过 Builder 对象来组装 Dialog 的各个部分，比如 title、buttons、message 等，将 Dialog 的构造和表示进行分离。

```java
AlertDialog

public class AlertDialog extends Dialog implements DialogInterface {
    // AlertController 接收 Builder 成员变量 P 中的各个参数
    private AlertController mAlert;
    
    protected AlertDialog(Context context) {
        this(context, resolveDialogTheme(context, 0), true);
    }

    protected AlertDialog(Context context, int theme) {
        this(context, theme, true);
    }

    AlertDialog(Context context, int theme, boolean createThemeContextWrapper) {
        super(context, resolveDialogTheme(context, theme), createThemeContextWrapper);

        mWindow.alwaysReadCloseOnTouchAttr();
        mAlert = new AlertController(getContext(), this, getWindow());
    }

    // 实际上调用的是 mAlert 的 setTitle 方法
	@Override
    public void setTitle(CharSequence title) {
        super.setTitle(title);
        mAlert.setTitle(title);
    }

    // 实际上调用的是 mAlert 的 setCustomTitle 方法
    public void setCustomTitle(View customTitleView) {
        mAlert.setCustomTitle(customTitleView);
    }
    
    public void setMessage(CharSequence message) {
        mAlert.setMessage(message);
    }

    public void setView(View view) {
        mAlert.setView(view);
    }

    // Builder 是 AlertDialog 的内部类
	public static class Builder {
		// 存储 AlertDialog 的各个参数，比如 title、message、icon 等
        private final AlertController.AlertParams P;
        private int mTheme;

        public Builder(Context context) {
            this(context, resolveDialogTheme(context, 0));
        }

		public Builder setTitle(int titleId) {
            P.mTitle = P.mContext.getText(titleId);
            return this;
        }
        
        public Builder setTitle(CharSequence title) {
            P.mTitle = title;
            return this;
        }

        public Builder setCustomTitle(View customTitleView) {
            P.mCustomTitleView = customTitleView;
            return this;
        }
        
        public Builder setMessage(int messageId) {
            P.mMessage = P.mContext.getText(messageId);
            return this;
        }

        // 构建 AlertDialog，传递参数，比如 title、message、icon 等
		public AlertDialog create() {
            // 这里 AlertDialog 是在 Builder 里创建的，
            // 在 Activity 里无法直接创建 AlertDialog
            final AlertDialog dialog = new AlertDialog(P.mContext, mTheme, false);
            // 把 P 的参数应用到 dialog 的 mAlert 里
            P.apply(dialog.mAlert);
            dialog.setCancelable(P.mCancelable);
            if (P.mCancelable) {
                dialog.setCanceledOnTouchOutside(true);
            }
            dialog.setOnCancelListener(P.mOnCancelListener);
            dialog.setOnDismissListener(P.mOnDismissListener);
            if (P.mOnKeyListener != null) {
                dialog.setOnKeyListener(P.mOnKeyListener);
            }
            return dialog;
        }
	}
}
```

外部类：AlertDialog 内部类：Builder
外部类：AlertController 内部类：AlertParams

title、message、icon 等首先赋值给 Builder，
Builder 又把参数赋值给 AlertParams，然后调用 `create` 方法，
会把 AlertParams 中的参数依次赋值给 AlertController，

### 注： ###

上面的流程并没有调用 AlertDialog 的 `setTitle` 等方法
当通过 `builder.create()` 方法创建一个 dialog 后，再调用 `setTitle` 等方法，才会调用 AlertDialog 的 `setTitle` 等方法。

在 `create` 方法内部，通过 `apply` 方法将 AlertParams 中的参数依次赋值给 AlertDialog。

```java
AlertController.AlertParams # apply

public void apply(AlertController dialog) {
    if (mCustomTitleView != null) {
        dialog.setCustomTitle(mCustomTitleView);
    } else {
        if (mTitle != null) {
            dialog.setTitle(mTitle);
        }
        if (mIcon != null) {
            dialog.setIcon(mIcon);
        }
        if (mIconId >= 0) {
            dialog.setIcon(mIconId);
        }
        if (mIconAttrId > 0) {
            dialog.setIcon(dialog.getIconAttributeResId(mIconAttrId));
        }
    }
    if (mMessage != null) {
        dialog.setMessage(mMessage);
    }
    if (mPositiveButtonText != null) {
        dialog.setButton(DialogInterface.BUTTON_POSITIVE, mPositiveButtonText,
                mPositiveButtonListener, null);
    }
    if (mNegativeButtonText != null) {
        dialog.setButton(DialogInterface.BUTTON_NEGATIVE, mNegativeButtonText,
                mNegativeButtonListener, null);
    }
    if (mNeutralButtonText != null) {
        dialog.setButton(DialogInterface.BUTTON_NEUTRAL, mNeutralButtonText,
                mNeutralButtonListener, null);
    }
    if (mForceInverseBackground) {
        dialog.setInverseBackgroundForced(true);
    }
    // 如果设置了 mItems，则表示是单选或者多选列表，此时创建一个 ListView
    if ((mItems != null) || (mCursor != null) || (mAdapter != null)) {
        createListView(dialog);
    }
    // 将 mView 设置给 Dialog
    if (mView != null) {
        if (mViewSpacingSpecified) {
            dialog.setView(mView, mViewSpacingLeft, mViewSpacingTop, mViewSpacingRight,
                    mViewSpacingBottom);
        } else {
            dialog.setView(mView);
        }
    } else if (mViewLayoutResId != 0) {
        dialog.setView(mViewLayoutResId);
    }

    /*
    dialog.setCancelable(mCancelable);
    dialog.setOnCancelListener(mOnCancelListener);
    if (mOnKeyListener != null) {
        dialog.setOnKeyListener(mOnKeyListener);
    }
    */
}
```

在 apply 函数中，只是将 AlertParams 参数设置到 AlertController 中。也就是说，只是完成了参数的设置，Dialog 还没有显示出来。接着通过 `show` 方法，就可以显示这个对话框：

```java
Dialog # show

public void show() {
    // 如果 Dialog 正在显示，就直接 return
    if (mShowing) {
        if (mDecor != null) {
            if (mWindow.hasFeature(Window.FEATURE_ACTION_BAR)) {
                mWindow.invalidatePanelMenu(Window.FEATURE_ACTION_BAR);
            }
            mDecor.setVisibility(View.VISIBLE);
        }
        return;
    }

    mCanceled = false;
    
    // 调用 onCreate
    if (!mCreated) {
        dispatchOnCreate(null);
    }

    onStart();
    // 获取 DecorView
    mDecor = mWindow.getDecorView();

    WindowManager.LayoutParams l = mWindow.getAttributes();
    if ((l.softInputMode
            & WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION) == 0) {
        WindowManager.LayoutParams nl = new WindowManager.LayoutParams();
        nl.copyFrom(l);
        nl.softInputMode |=
                WindowManager.LayoutParams.SOFT_INPUT_IS_FORWARD_NAVIGATION;
        l = nl;
    }

    try {
        // 将 mDeocr 添加到 WindowManager 中
        mWindowManager.addView(mDecor, l);
        mShowing = true;
        // 发送一个显示 Dialog 的消息
        sendShowMessage();
    } finally {
    }
}
```




```java
AlertController # setupContent

private void setupContent(LinearLayout contentPanel) {
    mScrollView = (ScrollView) mWindow.findViewById(R.id.scrollView);
    mScrollView.setFocusable(false);

    // Special case for users that only want to display a String
    // 如果用户只想显示一个 字符串
    mMessageView = (TextView) mWindow.findViewById(R.id.message);
    // 如果 mMessageView，就直接 return
    if (mMessageView == null) {
        return;
    }

    // 如果字符串不为空，就只显示一个字符串
    if (mMessage != null) {
        mMessageView.setText(mMessage);
    } else {
        // 如果字符串为空，就把 MessageView 隐藏
        mMessageView.setVisibility(View.GONE);
        // 从 ScrollView 中移除 MessageView
        mScrollView.removeView(mMessageView);

        // 如果 ListView 不为空（传递进来 mItems），就把 ScrollView 移除
        // 如果把 ListView 添加到 contentPanel 中
        if (mListView != null) {
            contentPanel.removeView(mWindow.findViewById(R.id.scrollView));
            contentPanel.addView(mListView,
                    new LinearLayout.LayoutParams(MATCH_PARENT, MATCH_PARENT));
            contentPanel.setLayoutParams(new LinearLayout.LayoutParams(MATCH_PARENT, 0, 1.0f));
        } else {
            // 如果 ListView 为空，就隐藏 contentPanel
            contentPanel.setVisibility(View.GONE);
        }
    }
}
```

疑惑：

```java
AlertDialog.Builder builder = new AlertDialog.Builder(this);
builder.setMessage("Message");
builder.setItems(new String[]{"a", "b"}, new DialogInterface.OnClickListener() {
    @Override
    public void onClick(DialogInterface dialog, int which) {

    }
}).show();
```

Message 和 ListView 都不显示