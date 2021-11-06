Retrofit 想必做 Android 开发的都用过，我们这篇文章来分析下它的原理。

在介绍原理之前，我们先介绍一下 Retrofit 中重要的几个对象，提前了解他们的作用，更有利于后续的源码解读。

### 动态代理

动态代理是可以在运行期动态创建某个 interface 的实例，我们定义好 Retrofit 接口之后，动态代理可以帮助我们创建出代理类，当我们调用接口的方法时，都会被 `InvocationHandler#invoke` 拦截,在这个方法中，我们可以知道被调用的是哪个方法，这个方法传入了什么参数，然后再做相应的处理。

### CallAdaper

在介绍 CallAdaper 之前，我们首先要讲下 Call 对象。Call 对象是什么？当我们调用一个 Retrofit 接口的时候，拿到的就是一个 Call 对象，Call 对象会帮助我们完成一个网络请求。

```kotlin
val service = retrofit.create(GithubService::class.java)
val repos: Call<List<Repo>> = service.listRepos("octocat")
repos.enqueue(object : Callback<List<Repo>?> {
    override fun onResponse(call: Call<List<Repo>?>, response: Response<List<Repo>?>) {
        
    }

    override fun onFailure(call: Call<List<Repo>?>, t: Throwable) {
    }
})
```

这里 repos 的类型就是 Call。

但是有的时候我们也需要自定义返回类型。比如 Retrofit 和 RxJava 相结合的时候，可能会返回 Observable 类型。这个时候 CallAdapter 就会发挥作用，帮助我们适配这些不同的返回类型，我们在定义 Retrofit 接口的时候，返回值是什么类型，CallAdapter#adapt 方法就会返回什么类型。

在 Retrofit 中，CallAdapter 是由 `CallAdapter.Factory` 的实现类生产出来的。

### Converter

从名字就可以看出，Converter 是用来做转换的。转换什么呢？转换服务器返回的数据类型。服务器返回的一般都是 json 字符串，我们拿到 json 字符串之后，需要把 json 字符串转换为实体类才方便后续的使用。有了 Converter，我们拿到的直接就是一个实体类对象，省去了手动转换的过程。

在 Retrofit 中，Converter 是由 `Converter.Factory` 的实现类生产出来的。