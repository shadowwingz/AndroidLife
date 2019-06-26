首先大致描述一下 AsyncTask 的工作原理：

AsyncTask 有两个线程池（`SERIAL_EXECUTOR` 和 `THREAD_POOL_EXECUTOR`）和一个 Handler（`InternalHandler`）
    
- 线程池 `SERIAL_EXECUTOR` 用于任务的排队， 
- 线程池 `THREAD_POOL_EXECUTOR` 用于真正的执行任务， 
- `InternalHandler` 用于将执行环境从线程池切换到主线程。

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
            return postResult(doInBackground(mParams));
        }
    };

    // 再次封装
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

AsyncTask # executeOnExecutor

public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
    ......

    // 初始化执行任务所需要的参数
    mWorker.mParams = params;
    // 执行任务
    exec.execute(mFuture);

    return this;
}
```

调用 `exec.execute(mFuture)`，就开始执行任务了。任务并不是立刻被执行，而是先在 sDefaultExecutor 线程池中排队，然后 sDefaultExecutor 线程池依次取出任务，交给 THREAD_POOL_EXECUTOR 线程池执行。

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

调用 `r.run()` 会执行 `postResult(doInBackground(mParams))`，doInBackground 方法是由我们开发者实现的，这里无需关注。我们再看 postResult 方法：

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

finish 方法的逻辑很简单，就是拿到任务执行结果，然后回调 onPostExecute 方法，onPostExecute 方法也是开发者实现的。

### 总结： ###

最后，总结一下，AsyncTask 会把任务投递给 SERIAL_EXECUTOR，SERIAL_EXECUTOR 会依次取出任务，交给 THREAD_POOL_EXECUTOR 去执行，THREAD_POOL_EXECUTOR 执行完了之后，会通过 InternalHandler 切换到主线程，然后回调 onPostExecute 方法。