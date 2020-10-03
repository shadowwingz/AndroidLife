<!-- TOC -->

- [LruCahce 是什么](#lrucahce-%E6%98%AF%E4%BB%80%E4%B9%88)
- [站在设计者的角度思考一下，如何设计 LRU 算法](#%E7%AB%99%E5%9C%A8%E8%AE%BE%E8%AE%A1%E8%80%85%E7%9A%84%E8%A7%92%E5%BA%A6%E6%80%9D%E8%80%83%E4%B8%80%E4%B8%8B%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1-lru-%E7%AE%97%E6%B3%95)
    - [选用合适的数据结构](#%E9%80%89%E7%94%A8%E5%90%88%E9%80%82%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
    - [容量的计算](#%E5%AE%B9%E9%87%8F%E7%9A%84%E8%AE%A1%E7%AE%97)
- [LruCache 原理解析](#lrucache-%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90)
    - [存元素](#%E5%AD%98%E5%85%83%E7%B4%A0)
    - [取元素](#%E5%8F%96%E5%85%83%E7%B4%A0)
- [相关问题](#%E7%9B%B8%E5%85%B3%E9%97%AE%E9%A2%98)
    - [LruCache 是线程安全的吗？](#lrucache-%E6%98%AF%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E5%90%97)

<!-- /TOC -->

## LruCahce 是什么

当我们从网络加载一张图片的时候，图片加载完成后会被缓存起来，后面再次加载这张图片的时候，就不用再从网络加载了，可以直接从缓存中取出图片来加载。

这样的话，就会存在一个问题，缓存也会占用手机的内存，而手机的内存是有限的，分配给 App 的内存就更加有限了。

比如手机内存是 4G，分配给 App 内存是 100M，这 100M 内存肯定不能全部用来当图片的缓存，因为 App 在运行的时候也要占用内存空间。

那分配多少呢？先分配 20M 用来当图片缓存吧。

OK，有了这 20M 图片缓存，当我们把图片链接加载为 Bitmap 后，就可以把这个 Bitmap 给存到图片缓存中了，这个很容易理解。

那我们不停的存 Bitmap，20M 缓存很快就顶不住了，可是我们还要继续存，那没办法，只能从缓存里删点图片了，删哪些呢？

一种简单的方法是在把图片存入缓存的时候记录下存入时间，删除的时候就把存入时间最早的图片给删了。

但是这样会有个问题，如果存入时间最早的图片最近又被从缓存中取出使用了，那这张图片显然不应该从缓存中删掉。

那我们改进一下，每次图片从缓存中取出的时候，我们都更新一下存入时间，把存入时间修改为取出的时间，

这样的话就不会有刚刚的问题了，如果最早存入缓存的图片近期又被使用了，那么这张图片的存入时间会发生改变，所以就不会被当作最早存入的图片被删掉。

这种 `当缓存空间满了的时候，将最近最少使用的数据从缓存空间中删除以增加可用的缓存空间来缓存新数据的算法` 就是 LRU 算法。

LRU 全称为 `Least Recently Used`，即最近最少使用。它是一种缓存算法。

## 站在设计者的角度思考一下，如何设计 LRU 算法

在研究 LruCache 的原理之前，我们先尝试自己设计一个 LRU 算法，这样能有助于理解 LruCache 的设计思想。

如果让我们自己设计一个 LRU Cache，我们应该怎么设计？

### 选用合适的数据结构

先来分析我们要存储的数据，首先，我们要存图片，肯定不是单纯的 `cache.put(bitmap)` 这样存，如果真这样存的话就不好取了，你怎么取出你想要的那个缓存呢？

所以我们得有个 key，这个 key 相当于 Bitmap 的别名，key 和 Bitmap 是绑定的，所以我们存 Bitmap 的时候就可以用 `cache.put(key, bitmap)` 来存，取的时候用 `cache.get(key)` 来取。

也就是 `key-value` 存储结构，由于 `key-value` 存储需要支持两个参数，那 List 类型的数据类型，比如 ArrayList、LinkedList，就被淘汰了，因为它们不是 `key-value` 结构。

支持 `key-value` 的数据结构，一般是 Map 类型。

但是 Map 也分好几种，比如最常见的 HashMap，还有 ArrayMap，LinkedHashMap。这又该怎么选呢？

这个后面说，在 Android 的 LruCache 中选用的是 LinkedHashMap。

### 容量的计算

假如我们的 Cache 是用来存储 10 个 Int 数字。那我们每存入一个数字，Cache 中已存储的数字个数就要加 1。这种情况下缓存关心的是个数。

假如我们的 Cache 是用来存储 20M 图片，那我们每存入一张图片，Cache 中已存储的图片内存就要加上新存入的图片的内存。这种情况下缓存关心的是内存大小。

当我们向 Cache 中存入数字或者图片时，我们需要有一种方式可以计算出存入的数字或者图片占了多大容量，这样我们才能和总容量做对比。

存入不同的元素，容量的计算方式是不同的。

对于数字来说，容量的计算很简单，就是它的个数，每存入一个数字，个数就 +1。

对于图片来说，容量的计算是图片的大小，也就是图片所占的内存。

这只是数字和图片，可能还有其它类型的元素，而我们作为设计者，显然是预料不到将来别人会用我们设计的 Cache 去存储什么类型元素。

所以我们应该把容量的计算交给调用者自己去实现。你传入的元素，容量该怎么计算你自己说了算。

我们只管接收最大容量，调用者每存入一个元素，我们就用调用者自定义的容量计算方式计算出这个元素所占的容量，然后判断已用容量有没有超过总容量。

## LruCache 原理解析

我们再来看下 Android 中的 LruCache 是怎么设计的。它的原理相对比较简单，主要分为`存`和`取`。

### 存元素

我们存储用的是 `put` 方法：

```java
public final V put(K key, V value) {
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }

    V previous;
    synchronized (this) {
        putCount++;
        size += safeSizeOf(key, value);
        // 将元素存入缓存
        previous = map.put(key, value);
        if (previous != null) {
            size -= safeSizeOf(key, previous);
        }
    }

    if (previous != null) {
        entryRemoved(false, key, previous, value);
    }

    // 判断已用缓存是否超出容量，如果超出就移除过期元素。
    trimToSize(maxSize);
    return previous;
}
```

put 方法的逻辑主要分为 2 步：

1. 将元素存入缓存
2. 判断已用缓存是否超出容量，如果超出就移除过期元素。

另外，put 方法中用到了 `safeSizeOf` 方法，也就是我们之前说的`容量的计算`：

```java
private int safeSizeOf(K key, V value) {
    int result = sizeOf(key, value);
    if (result < 0) {
        throw new IllegalStateException("Negative size: " + key + "=" + value);
    }
    return result;
}

protected int sizeOf(K key, V value) {
    return 1;
}
```

在 `safeSizeOf` 方法中会调用 `sizeOf` 方法来计算传入元素的容量大小，`sizeOf` 方法默认返回 1，1 就是元素的个数。

如果我们不想用元素的个数来当作元素的容量，比如我们传入 bitmap，那就要用图片的内存大小作为元素的容量了，这个时候我们可以重写 `sizeOf` 方法，使用 `bitmap.getAllocationByteCount()` 作为 Bitmap 的容量：

```java
@Override
protected int sizeOf(String key, Bitmap bitmap) {
    return bitmap.getAllocationByteCount();
}
```

### 取元素

从缓存中取元素用的是 `get` 方法：

```java
public final V get(K key) {
    if (key == null) {
        throw new NullPointerException("key == null");
    }

    V mapValue;
    synchronized (this) {
        // 从缓存中取出元素
        mapValue = map.get(key);
        if (mapValue != null) {
            hitCount++;
            return mapValue;
        }
        missCount++;
    }

    ......

    // 如果缓存中没有这个元素，就调用 create 方法创建这个元素
    // create 方法由开发者自己实现
    V createdValue = create(key);
    if (createdValue == null) {
        return null;
    }

    synchronized (this) {
        createCount++;
        // 将新创建的元素存入缓存
        mapValue = map.put(key, createdValue);

        if (mapValue != null) {
            // There was a conflict so undo that last put
            map.put(key, mapValue);
        } else {
            size += safeSizeOf(key, createdValue);
        }
    }

    if (mapValue != null) {
        entryRemoved(false, key, createdValue, mapValue);
        return mapValue;
    } else {
        // 判断已用缓存是否超出容量，如果超出就移除过期元素
        trimToSize(maxSize);
        return createdValue;
    }
}
```

get 方法主要分为 4 步：

1. 先获取缓存中 key 对应的元素，如果缓存中有元素，就直接返回
2. 如果缓存中没有元素，就调用 create 方法新创建一个元素。
3. 将新创建的元素存入缓存
4. 判断已用缓存是否超出容量，如果超出就移除过期元素。

## 相关问题

### LruCache 是线程安全的吗？

因为 LruCache 可能被多个线程同时操作，所以 put 方法和 get 方法需要加锁。

查看 put 方法和 get 方法，可以看到在存元素和取元素的时候，都有加锁。

```java
public final V put(K key, V value) {
    ......
    synchronized (this) {
        putCount++;
        size += safeSizeOf(key, value);
        previous = map.put(key, value);
        if (previous != null) {
            size -= safeSizeOf(key, previous);
        }
    }
    ......
}

public final V get(K key) {
    ......

    V mapValue;
    synchronized (this) {
        mapValue = map.get(key);
        if (mapValue != null) {
            hitCount++;
            return mapValue;
        }
        missCount++;
    }

    ......
}
```

所以 LruCache 是线程安全的。