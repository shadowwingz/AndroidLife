RemoteViews 的作用是在其他进程中显示并更新页面。先看一下它的构造方法：

```java
RemoteViews # RemoteViews

// packageName 是当前应用的包名
// layoutId 是待加载的布局文件
public RemoteViews(String packageName, int layoutId) {
    this(getApplicationInfo(packageName, UserHandle.myUserId()), layoutId);
}
```

第二个参数是布局文件的 id，这个布局文件将会被传递到其他进程并显示出来。RemoteViews 目前并不能支持所有的 View 类型，也就是说，只有支持的 View 类型才能被写在布局文件里，否则在其他进程加载布局文件的时候会报错。

它所支持的所有类型如下：

### Layout ###

FrameLayout、LinearLayout、RelativeLayout、GridLayout

### View ###

AnalogClock、Button、Chronometer、ImageButton、ImageView、ProgressBar、TextView、
ViewFlipper、ListView、GridView、StackView、AdapterViewFlipper、ViewStub

上面这些就是 RemoteViews 支持的 View 类型，它们的子类 RemoteViews 是【不支持】的。

之前说了，布局文件会被传递到其他进程并显示出来。这个“其他进程”是什么进程呢？

答案是系统的 SystemServer 进程。对于这个进程我们目前不需要知道太多，我们只需要知道这个进程是系统中很重要的一个进程，而我们也经常在和这个进程打交道，比如 Toast。

对于 TextView 来说，它的设置文字的方法是`setText`。很简单，直接传个字符串进来就完了。那 RemoteViews 呢？要是它里面有个 TextView，要给这个 TextView 设置点文字，直接调用 TextView 设置个文字，然后把布局传递到其他进程显示出来，完事。

这个方法不行，布局文件是没有序列化（实现 Parcelable）的，不能直接跨进程传输。

那 RemoteViews 内部是怎么做的呢？

如果你要给 TextView 设置文字，可以，调用下 `setTextViewText` 方法：

```java
RemoteViews # setTextViewText

// viewId 是被操作的 View 的 id，也就是 TextView 的 id
// setText 是方法名，也就是 TextView 设置文字的方法
// text 是要给 TextView 设置的文字
public void setTextViewText(int viewId, CharSequence text) {
    setCharSequence(viewId, "setText", text);
}
```

之前我们说了，RemoteViews 的构造方法的第二个参数是布局 id，这个布局 id 会被跨进程传递到系统进程 SystemServer，系统进程拿到这个 id 后，会加载这个 id 对应的布局，这个逻辑是在 RemoteViews 的 apply 方法中实现的。

```java
RemoteViews # apply

/** @hide */
public View apply(Context context, ViewGroup parent, OnClickHandler handler) {
    RemoteViews rvToApply = getRemoteViewsToApply(context);

    View result;
    ......

    LayoutInflater inflater = (LayoutInflater)
            context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);

    // Clone inflater so we load resources from correct context and
    // we don't add a filter to the static version returned by getSystemService.
    inflater = inflater.cloneInContext(inflationContext);
    inflater.setFilter(this);
    // 加载 RemoteViews 中的 layoutId 对应的布局文件
    result = inflater.inflate(rvToApply.getLayoutId(), parent, false);

    rvToApply.performApply(result, parent, handler);

    return result;
}
```

在加载完布局文件后，就要给 TextView 设置个文字了。刚刚我们是调用了 `setTextViewText` 方法来给 TextView 设置文字，在 `setTextViewText` 内部又调用了 `setCharSequence` 方法，看下 `setCharSequence` 这个方法：

```java
RemoteViews # setCharSequence

public void setCharSequence(int viewId, String methodName, CharSequence value) {
    addAction(new ReflectionAction(viewId, methodName, ReflectionAction.CHAR_SEQUENCE, value));
}
```

可以看到，它的内部调用了 `addAction` 方法，而且，还 `new` 了一个 ReflectionAction，再看看 `addAction` 方法，

```java
RemoteViews # addAction

private void addAction(Action a) {
    ......
    if (mActions == null) {
        mActions = new ArrayList<Action>();
    }
    mActions.add(a);

    // update the memory usage stats
    a.updateMemoryUsageEstimate(mMemoryUsageCounter);
}
```

可以看到，`addAction` 方法是比较简单的，就是创建了一个类型为 Action 的 ArrayList，然后把 ReflectionAction 加到了这个 ArrayList 中。

继续看下 `performApply` 方法：

```java
RemoteViews # performApply

private void performApply(View v, ViewGroup parent, OnClickHandler handler) {
    if (mActions != null) {
        handler = handler == null ? DEFAULT_ON_CLICK_HANDLER : handler;
        final int count = mActions.size();
        for (int i = 0; i < count; i++) {
            Action a = mActions.get(i);
            a.apply(v, parent, handler);
        }
    }
}
```

`performApply` 的逻辑很简单，就是遍历了一下 `mActions`，也就是刚刚说的那个类型为 Action 的 ArrayList，然后调用了 Action 的 `apply` 方法，嗯，看 `apply` 方法：

```java
RemoteViews # Action

private abstract static class Action implements Parcelable {
    public abstract void apply(View root, ViewGroup rootParent,
            OnClickHandler handler) throws ActionException;
}
```

我们发现两件事，
- 第一，Action 实现了 Parcelable 接口，也就是说，Action 是可以跨进程传递的。
- 第二，`apply` 方法是个抽象方法，而且 Action 并没有实现它。

刚刚我们好像遇见个 ReflectionAction？看看它的 `apply` 方法：

```java

@Override
public void apply(View root, ViewGroup rootParent, OnClickHandler handler) {
    final View view = root.findViewById(viewId);
    if (view == null) return;

    Class<?> param = getParameterType();
    if (param == null) {
        throw new ActionException("bad type: " + this.type);
    }

    try {
        getMethod(view, this.methodName, param).invoke(view, wrapArg(this.value));
    } catch (ActionException e) {
        throw e;
    } catch (Exception ex) {
        throw new ActionException(ex);
    }
}
```

我们从 ReflectionAction 的命名和实现可以看到，ReflectionAction 表示的是一个反射动作，还记得那个 `setText` 字符串吗，在 ReflectionAction 的 `apply` 方法中，会调用 `getMethod` 方法来得到反射所需的 Method 对象，从而调用 TextView 的 `setText` 方法。

到这里，逻辑基本上就清晰了。RemoteViews 会记录我们想要对布局文件执行的操作（Action），注意，仅仅是记录，并没有去执行。然后布局 id 会跨进程传递到系统进程 SystemServer，在系统进程中，会根据布局 id 加载出布局，然后把 RemoteViews 中记录的操作遍历执行。