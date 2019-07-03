首先大致描述一下 AsyncTask 的工作原理：

AsyncTask 有两个线程池（`SERIAL_EXECUTOR` 和 `THREAD_POOL_EXECUTOR`）和一个 Handler（`InternalHandler`）
    
- 线程池 `SERIAL_EXECUTOR` 用于任务的排队， 
- 线程池 `THREAD_POOL_EXECUTOR` 用于真正的执行任务， 
- `InternalHandler` 是一个关联了主线程 Looper 的 Handler，用于将执行环境从线程池切换到主线程。

我们创建一个 AsyncTask 对象，都会重写它的 doInBackground 方法，doInBackground 方法会被封装成一个 WorkerRunnable 对象，投递到任务队列中，线程池 `SERIAL_EXECUTOR` 会不停的从任务队列中取出任务，然后交给 `THREAD_POOL_EXECUTOR` 线程池去执行。

这篇文章分析的是下面代码的执行过程，也就是调用了 AsyncTask 的 execute 方法之后，AsyncTask 内部到底经历了怎样的流程，`onPreExecute`、`doInBackground`、`onPostExecute` 又是什么时候，在什么线程被调用的。

```java
new AsyncTask<Void, Void, Void>() {
    @Override
    protected void onPreExecute() {
        super.onPreExecute();
    }

    @Override
    protected Void doInBackground(Void... params) {
        return null;
    }

    @Override
    protected void onPostExecute(Void aVoid) {
        super.onPostExecute(aVoid);
    }
}.execute();
```

我们来梳理一下执行过程，首先，当我们调用创建 AsyncTask 对象并调用其 execute 方法时，会初始化执行任务所需要的参数，并将任务封装起来，以便线程池执行任务：

```java
AsyncTask # AsyncTask

public AsyncTask() {
    // 封装要执行的任务
    mWorker = new WorkerRunnable<Params, Result>() {
        public Result call() throws Exception {
            mTaskInvoked.set(true);

            Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
            //noinspection unchecked
            // doInBackground 方法被封装到 WorkerRunnable 类型的对象 mWorker 中
            return postResult(doInBackground(mParams));
        }
    };

    // mWorker 又再次封装被封装到 FutureTask 类型的对象 mFuture 中
    mFuture = new FutureTask<Result>(mWorker) {
        @Override
        protected void done() {
            try {
                postResultIfNotInvoked(get());
            } catch (InterruptedException e) {
                android.util.Log.w(LOG_TAG, e);
            } catch (ExecutionException e) {
                throw new RuntimeException("An error occured while executing doInBackground()",
                        e.getCause());
            } catch (CancellationException e) {
                postResultIfNotInvoked(null);
            }
        }
    };
}
```

可以看到，`doInBackground` 方法被封装到 WorkerRunnable 类型的对象 mWorker 中，mWorker 又再次封装被封装到 FutureTask 类型的对象 mFuture 中，我们可以猜到，最终就是这个 mFuture 对象会被调用。

调用 AsyncTask 的 execute 方法之后，最终会调用 `executeOnExecutor` 方法，`exec.execute(mFuture)`，就开始执行任务了。

```java
AsyncTask # executeOnExecutor

public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
    ......

    onPreExecute();
    // 初始化执行任务所需要的参数
    mWorker.mParams = params;
    // 执行任务
    exec.execute(mFuture);

    return this;
}
```

首先，onPreExecute 方法会被调用，那么 onPreExecute 方法是被执行在什么线程呢？这个要看 execute 方法是被执行在什么线程，我们一般是在主线程调用 AsyncTask 的 execute 方法，所以 onPreExecute 方法也一般是执行在主线程。

我们继续看，接着会执行 `exec.execute(mFuture)` 方法，mFuture 我们并不陌生，就是被封装起来的任务，也就是 `doInBackground` 方法。而 exec 是 AsyncTask 内部定义的线程池 SerialExecutor，这个线程池内部有一个队列，是专门用来存储任务的，线程池会一般不停的把任务丢到队列中，一般不停地从队列中取出任务执行。具体的执行是交给 THREAD_POOL_EXECUTOR 线程池去执行。

```java
AsyncTask # SerialExecutor

public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;

private static class SerialExecutor implements Executor {
    // 双端队列，用来装载所有要执行的任务（task）
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
    // 即将要执行的任务
    Runnable mActive;

    public synchronized void execute(final Runnable r) {
        // 把 Runnable 对象（实际上是 FutureTask 对象，FutureTask 是一个并发类，
        // 在这里它充当了 Runnable 的作用）再封装为 Runnable 对象
        // 这里只封装，不执行 Runnable ，Runnable 是在 THREAD_POOL_EXECUTOR 中执行的。
        // 虽然 THREAD_POOL_EXECUTOR 是并行执行任务，
        // 但是 SerialExecutor 是串行给 THREAD_POOL_EXECUTOR 提供任务，
        // 所以 THREAD_POOL_EXECUTOR 也串行执行任务。

        // 把 FutureTask 对象 插入到任务队列 mTasks 中
        mTasks.offer(new Runnable() {
            public void run() {
                try {
                    // THREAD_POOL_EXECUTOR 一旦执行任务，
                    // 就会调用 mActive 的 run 方法，
                    // 也就会调用 r 的 run 方法，这里的 r 是 FutureTask 对象，
                    // 查看 FutureTask 源码发现，调用 FutureTask 对象的 run 方法，
                    // 会调用 Callable 的 call 方法，
                    // 而 WorkerRunnable 实现了 Callable 接口，
                    // 所以会调用 WorkerRunnable 的 call 方法。
                    r.run();
                } finally {
                    // 一个 AsyncTask 任务执行完后，
                    // AsyncTask 会继续执行其他任务
                    // 直到所有的任务都被执行为止。
                    // 从这一点可以看出，在默认情况下，  
                    // AsyncTask 是串行执行的。
                    scheduleNext();
                }
            }
        });
        // 第一次调用 execute 方法时，mActive 为 null，
        // 所以会调用 scheduleNext 方法
        if (mActive == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        // 从双端队列中取出任务，如果任务不为 null，
        // 就让 THREAD_POOL_EXECUTOR 执行任务，
        // 也就是调用 mActive 的 run 方法。
        if ((mActive = mTasks.poll()) != null) {
            THREAD_POOL_EXECUTOR.execute(mActive);
        }
    }
}
```

线程池执行的任务是 `r.run()`，r 实际上是前面封装的 `mWorker`：

```java
AsyncTask # AsyncTask

mWorker = new WorkerRunnable<Params, Result>() {
    public Result call() throws Exception {
        mTaskInvoked.set(true);

        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        //noinspection unchecked
        return postResult(doInBackground(mParams));
    }
};
```

调用 `r.run()` 会执行 `postResult(doInBackground(mParams))`，doInBackground 方法就不说了，我们看下 postResult 方法：

```java
AsyncTask # postResult

private Result postResult(Result result) {
    @SuppressWarnings("unchecked")
    // 把执行的结果 result 封装到 AsyncTaskResult 中
    // 这里为什么要封装呢？
    // 因为每个任务和其结果是一一对应的。
    Message message = sHandler.obtainMessage(MESSAGE_POST_RESULT,
            new AsyncTaskResult<Result>(this, result));
    // 把消息发送给 sHandler，也就是 InternalHandler
    message.sendToTarget();
    return result;
}

AsyncTask # AsyncTaskResult

private static class AsyncTaskResult<Data> {
    final AsyncTask mTask;
    final Data[] mData;

    AsyncTaskResult(AsyncTask task, Data... data) {
        mTask = task;
        mData = data;
    }
}
```

再看 InternalHandler：

```java
AsyncTask # InternalHandler

private static class InternalHandler extends Handler {
    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        // 从 msg 中取出 AsyncTaskResult
        AsyncTaskResult result = (AsyncTaskResult) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                // 如果任务执行完了，就从 AsyncTaskResult 中取出
                // AsyncTask 对应的执行结果，然后调用
                // 对应 AsyncTask 的 finish 方法。
                // 这里就体现了每个任务和其结果一一对应的。
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}
```

InternalHandler 是用来切换线程的，我们的任务执行完后，要把执行结果回调到主线程，这时候就要用到 InternalHandler 了。

再看 finish 方法：

```java
AsyncTask # finish

private void finish(Result result) {
    // 判断 AsyncTask 有没有被取消，如果没有被取消，
    // 就回调 onPostExecute 方法
    if (isCancelled()) {
        onCancelled(result);
    } else {
        onPostExecute(result);
    }
    // 标记任务为已完成
    mStatus = Status.FINISHED;
}
```

finish 方法的逻辑很简单，就是拿到任务执行结果，然后回调 onPostExecute 方法，onPostExecute 方法也是开发者实现的。因为 onPostExecute 方法是在 finish 方法中被调用的，而 finish 方法是在 Handler 中被调用的，所以 onPostExecute 执行在主线程。

### 总结： ###

最后，总结一下，AsyncTask 会把任务投递给 SERIAL_EXECUTOR 的队列中，SERIAL_EXECUTOR 会不停从任务队列中取出任务，交给 THREAD_POOL_EXECUTOR 去执行，THREAD_POOL_EXECUTOR 执行完了之后，会通过 InternalHandler 切换到主线程，然后回调 onPostExecute 方法。

### 问题 ###

#### 如果在两个 Activity 中各 new 一个 AsyncTask，任务会排队执行吗？ ####

会，因为 AsyncTask 中的 `SerialExecutor` 和 `THREAD_POOL_EXECUTOR` 都是 static 的，所以即使 new 了两个 AsyncTask，但是线程池全局只有一个，所以任务会排队执行。

#### AsyncTask 执行的任务个数有限制吗？ ####

可以看 THREAD_POOL_EXECUTOR 的构造：

```java
public static final Executor THREAD_POOL_EXECUTOR
    = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
            TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
```

在 THREAD_POOL_EXECUTOR 的构造方法中有个 sPoolWorkQueue，它的大小就是 AsyncTask 任务的个数限制：

```java
private static final BlockingQueue<Runnable> sPoolWorkQueue =
    new LinkedBlockingQueue<Runnable>(128);
```

也就是 128。

#### AsyncTask 是串行还是并行，怎么实现并行 ####

Android 3.0 以上的AsyncTask 默认是串行执行任务的。

如果要并行，可以调用 executeOnExecutor 方法。

#### AsyncTask 可以自定义线程池吗？ ####

可以，调用 executeOnExecutor，传入自己实现的线程池即可。

