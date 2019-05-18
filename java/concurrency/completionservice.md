# CompletionService

实现原理：内部维护一个阻塞队列，任务执行结束就把执行结果加入到阻塞队列。

实现类是 **`ExecutorCompletionService`**：

```java
public class ExecutorCompletionService<V> implements CompletionService<V> {
    // 默认使用无界队列 LinkedBlockingQueue
    public ExecutorCompletionService(Executor executor)
    
    public ExecutorCompletionService(Executor executor, BlockingQueue<Future<V>> completionQueue)
}
```

`CompletionService` 有 5 个方法：

```java
public interface CompletionService<V> {
    Future<V> submit(Callable<V> task);

    // 与 ThreadPoolExecutor.submit(Runnable task, T result) 类似
    Future<V> submit(Runnable task, V result);

    // 若阻塞队列为空，则会阻塞
    Future<V> take() throws InterruptedException;

    // 若阻塞队列为空，则返回 null
    Future<V> poll();

    // 超时版的 poll 方法
    Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
}

```

