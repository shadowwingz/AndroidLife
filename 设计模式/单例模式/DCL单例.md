<!-- TOC -->

- [优化懒汉模式的效率](#优化懒汉模式的效率)
- [单例被重复创建问题](#单例被重复创建问题)
  - [解决方案：同步代码块内再进行一次判空](#解决方案同步代码块内再进行一次判空)
  - [为什么这样可以解决？](#为什么这样可以解决)
- [新的问题，JVM 重排序](#新的问题jvm-重排序)
  - [解决方案：volatile](#解决方案volatile)

<!-- /TOC -->

在 懒汉模式 中，我们讨论了懒汉模式的单例，发现懒汉模式的单例存在着各种局限性。

```java
懒汉模式

public class Singleton {

    private static Singleton instance;

    private Singleton() {

    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

用 synchronized 实现单例看上去不错，不过它的缺点也显而易见，就是效率低下，每次获取单例都要先获取到锁。

## 优化懒汉模式的效率

那么，怎么才能优化效率呢？

我们用伪代码来描述一下获取单例的流程：

> 获取锁
> 
> 执行 getInstance 方法
> 
> 对 instance 判空
> 
> 创建 Singleton 对象
> 
> 返回 instance
> 
> 释放锁

看到这个流程，我们发现，不管 instance 是不是 null，都要先获取锁，再释放锁。这就很没有必要了，如果 instance 不为 null，是没有必要加锁的，直接把 instance 返回就可以了。

所以，我们可以优化下流程：

> 执行 getInstance 方法
> 
> 对 instance 判空
> 
> 如果 instance 不为空，返回 instance 对象
> 
> 如果 instance 为空，获取锁
> 
> 创建 Singleton 对象
> 
> 返回 instance
> 
> 释放锁

优化后的流程里，我们对 instance 做了判断，只有 instance 为空的时候，才去获取锁。

由于单例本身也只有第一次获取的时候才需要创建对象，所以这样优化可以大大提升效率。

代码如下：

```java
public class Singleton {

    private static Singleton instance;

    private Singleton() {

    }

    public static Singleton getInstance() {
        // 1
        if (instance == null) {
            // 2
            synchronized (Singleton.class) {
                // 3
                instance = new Singleton();
            }
        }
        return instance;
    }
}
```

那么这样优化有没有什么问题呢？

有，代码 1 处的判断在多线程环境下不可靠，会导致 instance 被重复创建。

## 单例被重复创建问题

假如 A 线程执行了代码 1，判断结果为 instance 为 null，因此准备执行代码 2 获取锁，这个时候切换到线程 B，

线程 B 同样执行了代码 1，判断结果为 instance 为 null，然后执行代码 2，获取锁，然后执行代码 3 创建 instance 实例。

接着再切换回线程 A，由于刚刚判断 instance 为 null，因此线程 A 会去创建一个 instance 实例，但是刚刚线程 B 已经创建了 instance，因此线程 A 会重复创建。

### 解决方案：同步代码块内再进行一次判空

由于代码 2 处可能会切换线程，导致代码 1 处的判断结果不可靠。

因此我们需要在 synchronized 代码块内再对 instance 进行一次判空。

```java
public static Singleton getInstance() {
    // 1
    if (instance == null) {
        // 2
        synchronized (Singleton.class) {
            // 3
            if (instance == null) {
                // 4
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```

### 为什么这样可以解决？

我们再思考一下，为什么同步代码块外的判空不靠谱，再加个同步代码块内部的判空就行了呢？

首先，我们要知道同步代码块内就做了 2 件事，一个是判空，一个是创建单例。

由于同步代码块在同一时刻只能被一个线程执行，因此如果 A 线程在同步代码块中进行了判空，这个时候即使切换到 B 线程，B 线程也没法进入同步代码块内创建单例。所以单例的创建只能依靠 A 线程来创建。

## 新的问题，JVM 重排序

那这个单例还有问题吗？

有，还是 JVM 重排序问题。执行到代码 4 的时候，当 instance 指向 Singleton 对象，但是 Singleton 对象还没创建好，这时候切换线程，会导致代码 1 处的 if 语句不满于条件，因为 instance 会被返回回去，而 instance 对象实际上并没有创建好，所以使用的时候会出问题。

### 解决方案：volatile

解决方案的话，就是给 instance 加一个 volatile 修饰，禁止 JVM 重排序。

所以一个标准的 DCL 单例应该是这个样子：

```java
public class Singleton {

    private static volatile Singleton instance;

    private Singleton() {

    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```