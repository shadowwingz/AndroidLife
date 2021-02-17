我们如果想知道一个方法的执行耗时，最简单的方法就是先记录一下方法开始执行时的时间戳，再记录一下方法执行完毕后时的时间戳，拿这两个时间戳想减，得到的就是方法的执行耗时。

那如果我们想知道多个方法的执行耗时呢，我们就需要在每一个方法的执行前记录一下时间戳，在每一个方法的执行后再记录一下时间戳。

很明显，这种方法的工作量很大，而且每一个方法的代码都要修改。

那有没有方便一点的方法呢，如果我想记录 Activity 每个生命周期方法的执行耗时，那我需要把 onCreate、onStart、...、onXXX 都改一遍吗？

我们可以使用 AOP（Aspect Oriented Programming），面向切面编程。

关于 AOP 的简介这里不多说了，我们只需要知道，AOP 可以帮我们自动在方法的前后记录时间戳，并计算出方法的执行耗时，不需要我们手动在一个个方法前后加时间戳，可以大大节省我们的时间：

```java
@Aspect
public class ActivityAspect {

  @Around("execution( * com.shadowwingz.androidpractice.performance.aop.BaseActivity.**(..))")
  public void aopActivityAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
    Signature signature = joinPoint.getSignature();
    String name = signature.toShortString();
    long time = System.currentTimeMillis();
    joinPoint.proceed();
    Log.i("helloAOP", "aspect:::" + name + " cost " + (System.currentTimeMillis() - time));
  }
}
```

- @Aspect 注解表示这个文件是 AOP 文件
- ActivityAspect 类名可以随便起，能表达 AOP 要实现的功能就行
- @Around 注解可以同时在拦截方法的前后执行一段逻辑
- `execution( * com.shadowwingz.androidpractice.performance.aop.BaseActivity.**(..))` 表示拦截 BaseActivity 的方法
- ProceedingJoinPoint 具体拦截的方法