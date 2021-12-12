# 使用反射实现 ButterKnife

<!-- TOC -->

- [传统写法](#%E4%BC%A0%E7%BB%9F%E5%86%99%E6%B3%95)
- [开始改造](#%E5%BC%80%E5%A7%8B%E6%94%B9%E9%80%A0)
    - [自定义 BindView 注解](#%E8%87%AA%E5%AE%9A%E4%B9%89-bindview-%E6%B3%A8%E8%A7%A3)
    - [改造遇到的问题](#%E6%94%B9%E9%80%A0%E9%81%87%E5%88%B0%E7%9A%84%E9%97%AE%E9%A2%98)
    - [解决方案](#%E8%A7%A3%E5%86%B3%E6%96%B9%E6%A1%88)
- [抽取出一个 Libiary](#%E6%8A%BD%E5%8F%96%E5%87%BA%E4%B8%80%E4%B8%AA-libiary)

<!-- /TOC -->

## 传统写法

假如我们要在屏幕上写一个 TextView，我们的代码一般是这样的。

```java
public class MainActivity extends AppCompatActivity {

  // 1
  TextView tvShow;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    // 2
    tvShow = findViewById(R.id.tv_show);

    // 3
    tvShow.setText("我是测试文本");
  }
}
```

1 处的代码声明了一个 TextView 类型的变量 `tvShow`，2 处的代码创建了一个 TextView 对象并赋值给 `tvShow`，3 处给 `tvShow` 设置了一个文本值。

## 开始改造

既然我们要改造成 ButterKnife 的样式，那首先我们得把 1 处改成注解的样式：

```java
@BindView(R.id.tv_show)
TextView tvShow;
```

我们发现报红了。哦，对了，我们还没集成 ButterKnife 呢，啊，不对不对，我们是要自己写一个 ButterKnife！

### 自定义 BindView 注解

OK，那我们定义一个 BindView 注解：

```java
// 1
@Retention(RetentionPolicy.RUNTIME)
// 2
@Target(ElementType.FIELD)
// 3
public @interface BindView {
  // 4
  int value();
}
```

1 处我们给注解指定了一个 Retention，Retention 定义了注解被保留的时间长短。Retention 有三种类型：

> 1、SOURCE：让 BindView 注解只在 Java 源文件中存在，编译成 `.class` 文件 BindView 注解就不存在了。
> 
> 2、CLASS：让 BindView 注解不仅在 Java 源文件中存在，编译成 `.class` 文件后 BindView 也还在。
> 
> 3、RUNTIME：让 BindView 注解不仅在 Java 源文件和 `.class` 文件中存在，被类加载器加载到内存中也还在。

由于我们是要用反射来操作注解，而反射必须是在程序运行时才能使用，所以这里我们选择 `RetentionPolicy.RUNTIME`。

2 处 Target 注解决定 BindView 注解可以用在哪些成分上，比如用在类上，或者字段上，或者方法中。我们经常见到的注解 `@Override` 就是一个用在方法上的注解。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

这里我们只针对字段，所以使用 `ElementType.FIELD`。

3 处 `@interface` 是 Java 用来定义注解的语法，这个就不多说了。

4 处 value 是注解的默认值，当我们使用默认值时，我们使用 BindView 注解可以直接写 `@BindView(R.id.tv_show)`，如果我们把 value 改成 id：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface BindView {
  int id();
}
```

那我们使用 BindView 注解就得这么写：`@BindView(id = R.id.tv_show)`。

好了，到这里，BindView 注解我们就写完了。接下来，我们要使用这个注解来完成 findViewById。

我们新建一个类 Binding，然后把 findViewById 的逻辑转移到这个类中。

```java
public class Binding {
  public static void bind(MainActivity activity) {
    activity.tvShow = activity.findViewById(R.id.tv_show);
  }
}
```

然后修改一下 Activity 的代码：

```java
public class MainActivity extends AppCompatActivity {

  @BindView(R.id.tv_show)
  TextView tvShow;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    Binding.bind(this);

    tvShow.setText("我是测试文本");
  }
}
```

可以看到，我们用 `Binding.bind(this);` 取代了 findViewById。

### 改造遇到的问题

但是这样的 Binding 显然不是我们想要的，它还有不少问题：

1. bind 方法的形参类型是 MainActivity，而我们在实际使用中，是要在多个 Activity 中使用的，不能仅限于 MainActivity。
2. bind 方法中只对 `tv_show` 这个控件进行了 findViewById 操作，相当于把控件 id 给写死了。

### 解决方案

针对第一个问题，我们可以把形参改为 Activity，这样就不用受 Activity 类型的限制了。

针对第二个问题，我们可以遍历 Activity 的所有字段，找出有 BindView 注解的字段，然后取出注解值，再进行 findViewById 操作。

那我们来修改一下代码。

```java
public class Binding {
  public static void bind(Activity activity) {
    // 遍历 Activity 的所有字段
    for (Field field : activity.getClass().getDeclaredFields()) {
      // 找出有 BindView 的字段
      BindView bindView = field.getAnnotation(BindView.class);
      if (bindView != null) {
        // 取出 BindView 的值，比如 R.id.tv_show
        // 然后进行 findViewById 实例化控件
        View view = activity.findViewById(bindView.value());
        try {
          // 使用反射给控件赋值
          field.set(activity, view);
        } catch (IllegalAccessException e) {
          e.printStackTrace();
        }
      }
    }
  }
}
```

这样一来，我们就用反射实现了 ButterKnife。但是...好像还有点问题。

## 抽取出一个 Libiary

我们平常在使用第三方库的时候，都是以 libiary 的形式来使用。而我们现在是直接在主线程实现的，所以我们需要把 Binding 给移到一个单独的 libiary，然后让主工程集成这个 libiary。

我们新建一个叫 `lib-reflection` 的 libiary，然后修改 app 的 `build.gradle`，让 app 依赖这个 libiary：

```
dependencies {
    implementation project(":lib-reflection")
}
```

然后我们把 `BindView.java` 和 `Binding.java` 从 app 工程转移到 `lib-reflection` 这个 libiary。

我们运行一下，会发现 app 会报错：

```
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method 'void android.widget.TextView.setText(java.lang.CharSequence)' on a null object reference
```

看上去像是 TextView 没有成功的初始化。

除了这个错误之外，还会报一个错：

```
java.lang.IllegalAccessException: Class java.lang.Class<com.shadowwingz.lib_reflection.Binding> cannot access  field android.widget.TextView com.shadowwingz.mybutterknife.MainActivity.tvShow of class java.lang.Class<com.shadowwingz.mybutterknife.MainActivity>
```

这个错误是说，libiary 中的 Binding 类无法访问 app 工程的 MainActivity。

为什么无法访问呢？

因为主工程的 tvShow 没有用 public 修饰。之前我们的 MainActivity 和 Binding 类都是在同一个 module，同一个文件夹下，所以即使 tvShow 不是 public 也没有关系。

但是现在我们把 Binding 抽出，单独作为一个 module，这个时候 tvShow 就必须要改成 public，Binding 才能访问。

OK，那我们把 tvShow 改为 public。

```java
@BindView(R.id.tv_show)
public TextView tvShow;
```

可以正常运行。

但是，感觉有点不对...假如我们把这个 libiary 提供给别人用，难道还得要求别人定义字段一定要用 public？这明显很不合理。

那就换种方式解决吧，用 setAccessible 来绕过 Java 的权限控制，强制给字段赋值。

```java
try {
  // 绕过 Java 的权限控制，强制给字段赋值。
  field.setAccessible(true);
  // 使用反射给控件赋值
  field.set(activity, view);
} catch (IllegalAccessException e) {
  e.printStackTrace();
}
```

运行一下，我们发现，即使 tvShow 不是 public，也可以正常运行。

到这里，我们就完成了使用反射来实现 ButterKnife。