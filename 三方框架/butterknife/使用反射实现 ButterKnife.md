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

既然我们要改造成 ButterKnife 的样式，那首先我们得把 1 处改成注解的样式：

```java
@BindView(R.id.tv_show)
TextView tvShow;
```

我们发现报红了。哦，对了，我们还没集成 ButterKnife 呢，啊，不对不对，我们是要自己写一个 ButterKnife！

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