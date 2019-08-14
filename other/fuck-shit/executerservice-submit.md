# ExecuterService submit

## 起因

在线上排查问题时，请求不成功，但是日志里面没有找到任何异常。

## **重现**

写如下测试代码，发现 test1、test2 会直接抛出异常，test3 没有抛出异常，test4 在 16 行才抛出异常。

```java
private static ExecutorService es = Executors.newSingleThreadExecutor();

public static void main(String[] args) throws InterruptedException, ExecutionException {
    // test1
    new Thread(() -> { throw new RuntimeException("in thread start");}).start();
    
    // test2
    es.execute(() -> { throw new RuntimeException("in execute");});
    
    // test3
    es.submit(() -> { throw new RuntimeException("in submit");});
    
    // test4
    Future<Object> future = es.submit(() -> { throw new RuntimeException("in submit"); });
    Thread.sleep(1000);
    future.get();
    
    es.shutdown();
}

// test1
Exception in thread "Thread-0" java.lang.RuntimeException: in thread start
	at com.zhaoyun.se.fuckshit.ExecutorServiceSubmit.lambda$main$0(ExecutorServiceSubmit.java:18)
	at java.base/java.lang.Thread.run(Thread.java:834)
	
// test2
Exception in thread "pool-1-thread-1" java.lang.RuntimeException: in execute
	at com.zhaoyun.se.fuckshit.ExecutorServiceSubmit.lambda$main$1(ExecutorServiceSubmit.java:21)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:834)
	
// test4 16行 future.get()
Exception in thread "main" java.util.concurrent.ExecutionException: java.lang.RuntimeException: in submit
	at java.base/java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.base/java.util.concurrent.FutureTask.get(FutureTask.java:191)
	at com.zhaoyun.se.fuckshit.ExecutorServiceSubmit.main(ExecutorServiceSubmit.java:29)
Caused by: java.lang.RuntimeException: in submit
	at com.zhaoyun.se.fuckshit.ExecutorServiceSubmit.lambda$main$3(ExecutorServiceSubmit.java:27)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:834)

```

## **原因**

### **execute**

使用 ThreadPoolExecutor.execute 执行任务时，任务原封不动提交给 Worker，若任务抛出异常，Worker 会直接再次抛出：

```java
// java.util.concurrent
public class ThreadPoolExecutor extends AbstractExecutorService {
    private final class Worker extends AbstractQueuedSynchronizer implements Runnable {
        public void run() {
            runWorker(this);
        }
    }
    
    final void runWorker(Worker w) {
        ...
        try {
            while (task != null || (task = getTask()) != null) {
                ...
                try {
                    beforeExecute(wt, task);
                    try {
                        task.run();
                        afterExecute(task, null);
                    } catch (Throwable ex) {
                        afterExecute(task, ex);
                        throw ex; // 再次抛出
                    }
                } finally {
                    ...
                }
            }
            completedAbruptly = false;
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
}
```

### submit

使用 ThreadPoolExecutor.submit 执行任务时，任务会被封装成 FutureTask，FutureTask 的 run 方法会捕获异常，不会再次抛出，在 get 时才会抛出：

```java
// java.util.concurrent
public class ThreadPoolExecutor extends AbstractExecutorService {
    public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);
        return ftask;
    }
    
    protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
}

// java.util.concurrent
public class FutureTask<V> implements RunnableFuture<V> {
    public void run() {
        ...
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                ...
                try {
                    ...
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                ...
            }
        } finally {
            ...
        }
    }
}
```



