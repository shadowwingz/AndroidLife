LayoutInflater 想必大家都不陌生，我们加载布局的时候用的就是它。先回忆下我们是怎么使用 LayoutInflater 的。

```java
LayoutInflater inflater = LayoutInflater.from(context);
// resourceId 是要加载的布局 id
// root 是该布局的父布局
inflater.inflate(resourceId, root)
```

首先，使用 `LayoutInflater.from(context)` 获取到 LayoutInflater 对象，然后调用 LayoutInflater 的 `inflate` 方法去加载布局就行了。这里要注意，`inflate` 有两个参数，第一个参数是要加载的布局 id，第二个参数是该布局的父布局。

我们来试一下，首先新建一个 `LayoutInflaterActivity`，然后修改 `LayoutInflaterActivity` 的布局文件 `activity_layout_inflater.xml`，给 LinearLayout 加一个 id，方便等下  `LayoutInflaterActivity` 中通过 findViewById 来找到这个 LinearLayout。代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

</LinearLayout>
```

新建一个 button_layout.xml，代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Button xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button"
        android:textAllCaps="false">

</Button>
```

这个布局文件很简单，只有一个按钮。接下来，我们就要使用 LayoutInflater 把这个按钮添加到 Activity 的 LinearLayout 中。刚刚我们说了，有一个 inflate 方法可以加载布局，我们试下这个方法，修改 LayoutInflaterActivity：

```java
public class LayoutInflaterActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_inflater);
        ViewGroup mainLayout = (ViewGroup) findViewById(R.id.main_layout);
        LayoutInflater inflater = LayoutInflater.from(this);
        inflater.inflate(R.layout.button_layout, mainLayout);
    }
}
```

我们先实例化了 LinearLayout，也就是 mainLayout。然后获取了 LayoutInflater 的实例，最后调用 inflate 方法加载了布局。

运行一下项目：

![](https://raw.githubusercontent.com/shadowwingz/AndroidLife/master/art/LayoutInflater%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/1.png)

可以看到，Button 已经被成功的显示在界面上了。

接着，我们来看下 Button 是怎么被加载出来的。先看下 `inflate` 方法，调用 `inflate` 后，最终会进入 `inflate(parser, root, attachToRoot)` 方法中：

```java
public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
    synchronized (mConstructorArgs) {
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");

        final AttributeSet attrs = Xml.asAttributeSet(parser);
        Context lastContext = (Context)mConstructorArgs[0];
        mConstructorArgs[0] = mContext;
        View result = root;

        try {
            // Look for the root node.
            int type;
            while ((type = parser.next()) != XmlPullParser.START_TAG &&
                    type != XmlPullParser.END_DOCUMENT) {
                // Empty
            }

            if (type != XmlPullParser.START_TAG) {
                throw new InflateException(parser.getPositionDescription()
                        + ": No start tag found!");
            }

            final String name = parser.getName();
            
            if (DEBUG) {
                System.out.println("**************************");
                System.out.println("Creating root view: "
                        + name);
                System.out.println("**************************");
            }

            if (TAG_MERGE.equals(name)) {
                if (root == null || !attachToRoot) {
                    throw new InflateException("<merge /> can be used only with a valid "
                            + "ViewGroup root and attachToRoot=true");
                }

                rInflate(parser, root, attrs, false, false);
            } else {
                // Temp is the root view that was found in the xml
                final View temp = createViewFromTag(root, name, attrs, false);

                ViewGroup.LayoutParams params = null;

                if (root != null) {
                    if (DEBUG) {
                        System.out.println("Creating params from root: " +
                                root);
                    }
                    // Create layout params that match root, if supplied
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        // Set the layout params for temp if we are not
                        // attaching. (If we are, we use addView, below)
                        temp.setLayoutParams(params);
                    }
                }

                if (DEBUG) {
                    System.out.println("-----> start inflating children");
                }
                // Inflate all children under temp
                rInflate(parser, temp, attrs, true, true);
                if (DEBUG) {
                    System.out.println("-----> done inflating children");
                }

                // We are supposed to attach all the views we found (int temp)
                // to root. Do that now.
                if (root != null && attachToRoot) {
                    root.addView(temp, params);
                }

                // Decide whether to return the root that was passed in or the
                // top view found in xml.
                if (root == null || !attachToRoot) {
                    result = temp;
                }
            }

        } catch (XmlPullParserException e) {
            InflateException ex = new InflateException(e.getMessage());
            ex.initCause(e);
            throw ex;
        } catch (IOException e) {
            InflateException ex = new InflateException(
                    parser.getPositionDescription()
                    + ": " + e.getMessage());
            ex.initCause(e);
            throw ex;
        } finally {
            // Don't retain static reference on context.
            mConstructorArgs[0] = lastContext;
            mConstructorArgs[1] = null;
        }

        Trace.traceEnd(Trace.TRACE_TAG_VIEW);

        return result;
    }
}
```