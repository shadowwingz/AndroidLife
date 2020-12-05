<!-- TOC -->

- [前言](#前言)
- [synchronized 的作用](#synchronized-的作用)
- [synchronized 修饰静态方法](#synchronized-修饰静态方法)
- [synchronized 修饰代码块](#synchronized-修饰代码块)

<!-- /TOC -->

## 前言

我们在日常开发中，免不得要和多线程打交道，一说到多线程，我们第一个想到的应该就是 `synchronized`。

这篇文章中我们来研究一下 synchronized。

## synchronized 的作用

在说到 synchronized 的作用之前，我们先来看一段代码。

```java
public class NormalMethods {

    public static void main(String[] args) {
        NormalMethods methods1 = new NormalMethods();

        Thread t1 = new Thread(new Runnable() {
            @Override
            public void run() {
                methods1.printLog();
            }
        });

        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                methods1.printLog();
            }
        });

        t1.start();
        t2.start();
    }

    public void printLog() {
        try {
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + " is printing " + i);
                Thread.sleep(300);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

这段代码的逻辑是，创建 2 个子线程，然后分别打印线程名称和 i 的值。

输出如下：

```
Thread-0 is printing 0
Thread-1 is printing 0
Thread-0 is printing 1
Thread-1 is printing 1
Thread-1 is printing 2
Thread-0 is printing 2
Thread-0 is printing 3
Thread-1 is printing 3
Thread-0 is printing 4
Thread-1 is printing 4
```

从输出结果中我们可以看到，Thread-0 和 Thread-1 是交替打印的。之所以交替打印，是因为 Thread-0 和 Thread-1 是随机获取 CPU 时间片的，所以 CPU 可能先切换到 Thread-0，打印一下，再切换到 Thread-1，再打印一下。

那有没有一种方式，可以让 Thread-0 打印的时候，不让 Thread-1 打印呢？换句话说，让 Thread-0 先把从 0 到 4 打印完，再让 Thread-1 从 0 到 4 打印。

我们知道，Java 代码都是在 JVM（Java 虚拟机）中执行的，那是不是说，只要我们能在打印方法的前面，加一个标志位，当 JVM 执行到我们的打印方法的时候，看到了这个标志位，就不让其他线程来执行，让其他的线程先等一等，等我们的打印方法执行完了之后，我们把这个标志位给移除掉，这样就能让其他的线程来执行这段代码了。类似这样：

```java
线程执行到这里的时候，在这里放置一个标志位，有了这个标志位，虚拟机就不让其他的线程来执行这段代码

public void printLog() {
    try {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " is printing " + i);
            Thread.sleep(300);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

线程执行到这里的时候，移除掉标志位，标志位被移除，虚拟机才允许其他的线程来执行这段代码
```

这个标志位就是我们要讲的 synchronized。

OK，我们来试一下：

```java
// synchronized 
public synchronized void printLog() {
    try {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " is printing " + i);
            Thread.sleep(300);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

输出：

```
Thread-0 is printing 0
Thread-0 is printing 1
Thread-0 is printing 2
Thread-0 is printing 3
Thread-0 is printing 4
Thread-1 is printing 0
Thread-1 is printing 1
Thread-1 is printing 2
Thread-1 is printing 3
Thread-1 is printing 4
```

可以看到，现在的执行顺序是，Thread-0 先打印完，Thread-1 再打印。

synchronized 是一种锁，当 JVM 执行到 synchronized 修饰的方法的时候，线程先尝试获取锁，如果可以获取到锁，就执行对应的方法，如果获取不到锁，线程就阻塞住，等待其他线程释放锁。

我们刚刚是把 synchronized 加在一个实例方法 printLog 上，那 synchronized 还能不能加到其它的地方呢？

可以。synchronized 可以修饰静态类方法，还可以修饰代码块。

## synchronized 修饰静态方法

我们把 printLog 改成静态方法：

```java
public static synchronized void printLog() {
    try {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " is printing " + i);
            Thread.sleep(300);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

打印：

```
Thread-0 is printing 0
Thread-0 is printing 1
Thread-0 is printing 2
Thread-0 is printing 3
Thread-0 is printing 4
Thread-1 is printing 0
Thread-1 is printing 1
Thread-1 is printing 2
Thread-1 is printing 3
Thread-1 is printing 4
```

可以看到，Thread-0 先执行完，Thread-1 才接着执行。

## synchronized 修饰代码块

我们再来试一下 synchronized 修饰代码块：

```java
public void printLog() {
    try {
        synchronized (mLock) {
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() + " is printing " + i);
                Thread.sleep(300);
            }
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

输出结果和上面一样，就不贴了。