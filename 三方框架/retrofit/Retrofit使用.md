Retrofit 毫无疑问是目前最流行的网络请求框架，这里简单介绍一下 Retrofit 的使用。

使用 Retrofit 大概需要 7 个步骤：

1. 添加 Retrofit 库的依赖
2. 创建接收服务器返回数据的类
3. 创建用于描述网络接口的请求
4. 创建 Retrofit 实例
5. 创建网络请求接口实例
6. 发送网络请求
7. 处理返回数据

接下来依次说明这 7 个步骤。

### 添加 Retrofit 库的依赖

要使用一个框架，当然得先添加它的依赖：

```
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.0.2'
    // Retrofit库
    implementation 'com.squareup.okhttp3:okhttp:3.1.2'
    // Okhttp库
}
```

我们不仅添加了 Retrofit 的依赖库，还添加了 OkHttp 的依赖库。这是因为网络请求实际是由 OkHttp 来完成的，Retrofit 只是让我们更方便的使用 OkHttp。

### 创建接收服务器返回数据的类

这里我们创建一个实体类，用来接收服务器返回的数据。

```kotlin
data class Repo(
  var allow_forking: Boolean,
  var archive_url: String,
  var archived: Boolean,
  var assignees_url: String,
  var blobs_url: String,
  var branches_url: String,
  var clone_url: String,
  var collaborators_url: String,
  var comments_url: String,
  var commits_url: String,
  var compare_url: String,
  var contents_url: String,
  var contributors_url: String,
  var created_at: String,
  var default_branch: String,
  var deployments_url: String,
  var description: String,
  var disabled: Boolean,
  var downloads_url: String,
  var events_url: String,
  var fork: Boolean,
  var forks: Int,
  var forks_count: Int,
  var forks_url: String,
  var full_name: String,
  var git_commits_url: String,
  var git_refs_url: String,
  var git_tags_url: String,
  var git_url: String,
  var has_downloads: Boolean,
  var has_issues: Boolean,
  var has_pages: Boolean,
  var has_projects: Boolean,
  var has_wiki: Boolean,
  var homepage: String,
  var hooks_url: String,
  var html_url: String,
  var id: Int,
  var is_template: Boolean,
  var issue_comment_url: String,
  var issue_events_url: String,
  var issues_url: String,
  var keys_url: String,
  var labels_url: String,
  var language: Any,
  var languages_url: String,
  var license: Any,
  var merges_url: String,
  var milestones_url: String,
  var mirror_url: Any,
  var name: String,
  var node_id: String,
  var notifications_url: String,
  var open_issues: Int,
  var open_issues_count: Int,
  var owner: Owner,
  var `private`: Boolean,
  var pulls_url: String,
  var pushed_at: String,
  var releases_url: String,
  var size: Int,
  var ssh_url: String,
  var stargazers_count: Int,
  var stargazers_url: String,
  var statuses_url: String,
  var subscribers_url: String,
  var subscription_url: String,
  var svn_url: String,
  var tags_url: String,
  var teams_url: String,
  var topics: List<Any>,
  var trees_url: String,
  var updated_at: String,
  var url: String,
  var visibility: String,
  var watchers: Int,
  var watchers_count: Int
) {
  data class Owner(
    var avatar_url: String,
    var events_url: String,
    var followers_url: String,
    var following_url: String,
    var gists_url: String,
    var gravatar_id: String,
    var html_url: String,
    var id: Int,
    var login: String,
    var node_id: String,
    var organizations_url: String,
    var received_events_url: String,
    var repos_url: String,
    var site_admin: Boolean,
    var starred_url: String,
    var subscriptions_url: String,
    var type: String,
    var url: String
  )
}
```

### 创建用于描述网络接口的请求

```kotlin
interface GithubService {
  @GET("users/{user}/repos")
  fun listRepos(@Path("user") user: String) : Call<List<Repo>>
}
```

代码含义如下：

![](images/Retrofit%20使用.jpg)


### 创建 Retrofit 实例

```kotlin
val retrofit = Retrofit
      .Builder()
      .baseUrl("https://api.github.com") // 设置网络请求的 baseUrl
      .addConverterFactory(GsonConverterFactory.create()) // 设置数据解析器，把接口返回的数据转换为 Repo 对象
      .build()
```

这里解释一下数据解析器，当我们请求一个接口的时候，接口返回给我们的是 Json 数据，我们还需要把 Json 数据手动转换为实体类对象才能方便使用，当然，现在已经有不少的第三方框架可以帮助我们转换了，比如 GSON，但是每次请求之后都要使用 GSON 来转换一下总归是有些不爽。

Retrofit 考虑到了这一点，在创建 Retrofit 实例的时候调用 `addConverterFactory(GsonConverterFactory.create())` 设置数据解析器之后，我们使用 Retrofit 获取网络数据，拿到的就是实体类对象，省去了数据转换的功夫。


### 创建网络请求接口实例

```kotlin
// 创建网络请求接口的实例
val service = retrofit.create(GithubService::class.java)

// 对发送请求进行封装
val repos: Call<List<Repo>> = service.listRepos("octocat")
```

这里需要注意的是，调用 `service.listRepos("octocat")` 并不是发起网络请求，而是对于发送请求进行封装，从而创建出一个用来发送网络请求的 Call 对象。真正发起网络请求是由这 Call 对象来完成的。
### 发送网络请求

```kotlin
repos.enqueue(object : Callback<List<Repo>?> {
    // 请求成功回调
    override fun onResponse(call: Call<List<Repo>?>, response: Response<List<Repo>?>) {
        // response 是返回数据
    }

    // 请求失败回调
    override fun onFailure(call: Call<List<Repo>?>, t: Throwable) {
    }
})
```

### 处理返回数据

```kotlin
repos.enqueue(object : Callback<List<Repo>?> {
    override fun onResponse(call: Call<List<Repo>?>, response: Response<List<Repo>?>) {
        // 这里简单起见，只打印 response 中第一条数据的 name 的值
        println("Response: ${response.body()!![0].name}")
    }

    override fun onFailure(call: Call<List<Repo>?>, t: Throwable) {
    }
})
```

输出：

```
Response: boysenberry-repo-1
```