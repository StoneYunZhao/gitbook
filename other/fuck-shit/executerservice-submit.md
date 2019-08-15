# ExecuterService submit

## 起因

在线上排查问题时，请求不成功，但是日志里面没有找到任何异常。

## **重现**

写如下测试代码，发现 test1、test2 会直接抛出异常，**test3 没有抛出异常**，test4 在 16 行才抛出异常。

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
```

输出如下：

```text
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
                    setException(ex); // 捕获异常，不会直接抛出
                }
                ...
            }
        } finally {
            ...
        }
    }
    
    // 在 get 时才抛出异常
    public V get() throws InterruptedException, ExecutionException {
        ...
        return report(s);
    }
    
    private V report(int s) throws ExecutionException {
        Object x = outcome;
        if (s == NORMAL)
            return (V)x;
        if (s >= CANCELLED)
            throw new CancellationException();
        throw new ExecutionException((Throwable)x); // 抛出
    } 
}
```

## 解决方案

### 方案1 - 选择 execute

若提交任务后，不需要获取返回结果，则尽量使用 execute 方法，这样就能在标准输出打印异常栈。

**缺点**：只能在标准输出打印异常栈，不能在日志系统里面打印，所以日志文件中还是没有异常输出。

### 方案2 - 从任务出发

自定义任务接口 Task，所有任务都实现 Task：

```java
public final class SolutionByTask {
    private static ExecutorService es = Executors.newSingleThreadExecutor();

    public static void main(String[] args) throws InterruptedException {
        // test2
        Task t2 = () -> { throw new RuntimeException("in execute");};
        es.execute(t2);
        
        // test3
        Task t3 = () -> { throw new RuntimeException("in submit");};
        es.submit(t3);

        es.shutdown();
    }

    private interface Task extends Runnable {
        @Override
        default void run(){
            try {
                doRun();
            } catch (Exception e) {
                System.out.println("Task logger: " + e); // 在生产中可替换为其它任何你想做的事，比如日志组件
                throw e;
            }
        }

        void doRun();
    }
}
```

输出如下：

```text
// Task 接口日志组件打印的异常
Task logger: java.lang.RuntimeException: in execute

// JVM 标准输出打印的异常
Exception in thread "pool-1-thread-1" java.lang.RuntimeException: in execute
	at com.zhaoyun.se.fuckshit.SolutionByTask.lambda$main$0(SolutionByTask.java:15)
	at com.zhaoyun.se.fuckshit.SolutionByTask$Task.run(SolutionByTask.java:31)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:834)
	
// Task 接口日志组件打印的异常
Task logger: java.lang.RuntimeException: in submit
```

###  方案3 - 从线程池出发

自定义线程池继承 ThreadPoolExecutor，重写 afterExecute 方法：

```java
public final class SolutionByThreadPool {
    private static ExecutorService es = new MyThreadPool(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());

    public static void main(String[] args) throws InterruptedException {
        // test2
        es.execute(() -> { throw new RuntimeException("in execute");});

        // test3
        es.submit(() -> { throw new RuntimeException("in submit");});

        es.shutdown();
    }

    private static class MyThreadPool extends ThreadPoolExecutor {

        public MyThreadPool(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
            super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
        }

        @Override
        protected void afterExecute(Runnable r, Throwable t) {
            super.afterExecute(r, t);

            if (r instanceof FutureTask<?>) {
                try {
                    Future<?> f = (Future<?>) r;
                    if (f.isDone()) {
                        f.get();
                    }
                } catch (CancellationException ce) {
                    t = ce;
                } catch (ExecutionException ee) {
                    t = ee.getCause();
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }

            }

            if (t != null) {
                System.out.println("MyThreadPool logger: " + t);
            }
        }
    }
}
```

输出如下：

```text
// MyThreadPool 日志组件打印的异常
MyThreadPool logger: java.lang.RuntimeException: in execute

// JVM 标准输出打印的异常
Exception in thread "pool-1-thread-1" java.lang.RuntimeException: in execute
	at com.zhaoyun.se.fuckshit.SolutionByThreadPool.lambda$main$0(SolutionByThreadPool.java:14)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:834)
	
// MyThreadPool 日志组件打印的异常
MyThreadPool logger: java.lang.RuntimeException: in submit
```

