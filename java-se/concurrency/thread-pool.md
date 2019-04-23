# Thread Pool

Java 创建线程需要调用操作系统内核 API，然后操作系统分配一系列资源，所以**创建线程是一个重量级操作，应该避免频繁创建于销毁**。所以需要线程池来解决上述问题。

## ThreadPoolExecutor

### 构造函数

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler)
```

* corePoolSize：最小线程数。
* maximumPoolSize：最大线程数。
* keepAliveTiTime & unit：若一个线程空闲了一段时间，并且线程数量大于 corePoolSize，那么空闲的线程就能被回收。
* workQueue：阻塞的工作队列。
* threadFactory：自定义如何创建线程，可以指定有意义的线程名字
* handler：工作队列满了后，提交任务时的拒绝策略。
  * CallerRunsPolicy：提交任务的线程自己去执行。
  * AbortPolicy：默认，直接拒绝，抛出 RejectedExecutionException。
  * DiscardPolicy：直接丢弃。
  * DiscardOldestPolicy：丢弃最老的任务。

{% hint style="warning" %}
**默认拒绝策略要慎重使用**。如果线程池处理的任务非常重要，建议自定义自己的拒绝策略；并且在实际工作中，自定义的拒绝策略往往和**降级策略**配合使用。
{% endhint %}

### 核心逻辑

ThreadPoolExecutor 有个内部类 Worker，轮询去任务队列拿任务执行。

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
    private final class Worker 
        extends AbstractQueuedSynchronizer implements Runnable { 
               
        public void run() {
            runWorker(this);
        }
    }
    
    final void runWorker(Worker w) {
        try {
            while (task = getTask() != null) {
                try {
                    task.run();
                } finally {
                    task = null;
                }
            }
        } finally {
            processWorkerExit(w, completedAbruptly);
        }
    }
}
```

## Executors

Java 并发包提供了 Executors 可以快速创建线程池。

{% hint style="danger" %}
目前大厂的编码规范中基本上都**不建议使用 Executors** 了。因为 Executors 提供的很多方法默认使用的都是**无界**的 LinkedBlockingQueue。
{% endhint %}

