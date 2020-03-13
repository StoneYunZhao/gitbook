# Utility Class

## Semaphore

`java.util.concurrent.Semaphore`是对[信号量模型](preface.md#xin-hao-liang)的实现。

用于控制同时访问的线程个数。

底层基于 AQS，state 变量存储的是可用的剩余资源。

```java
/**
* 5台机器，8个工人，一台机器只能被一个工人使用。
*/
public class Test {
    public static void main(String[] args) {
        int N = 8;            //工人数
        Semaphore semaphore = new Semaphore(5); //机器数目
        for(int i=0;i<N;i++)
            new Worker(i,semaphore).start();
    }
 
    static class Worker extends Thread{
        private int num;
        private Semaphore semaphore;
        public Worker(int num,Semaphore semaphore){
            this.num = num;
            this.semaphore = semaphore;
        }
 
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println("工人"+this.num+"占用一个机器在生产...");
                Thread.sleep(2000);
                System.out.println("工人"+this.num+"释放出机器");
                semaphore.release();           
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## CountDownLatch

**主要用来解决一个线程等待多个线程的场景。**

计数器不能循环利用。当计数器减到0时，再调用 await 会直接通过。

底层用 AQS 实现，state 变量即为设置的 N。

## CyclicBarrier

**CyclicBarrier 是一组线程之间互相等待。**

计数器可以循环利用，而且能够自动重置，一旦计数器减到0，会自动重置到你设置的初始值。

## ThreadPoolExecutor

Java 创建线程需要调用操作系统内核 API，然后操作系统分配一系列资源，所以**创建线程是一个重量级操作，应该避免频繁创建于销毁**。所以需要线程池来解决上述问题。

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

### submit

`ThreadPoolExecutor` 的 `execute()` 方法不能取得任务结果，但是 `submit()` 方法可以。

```java
// Runnable 的 run 方法没有返回值，所以这个方法仅能用来判断任务是否结束
Future<?> submit(Runnable task)

// 假设这个方法返回 f，f.get() 的返回值就是传给 submit() 方法的参数 result。
<T> Future<T> submit(Runnable task, T result)

<T> Future<T> submit(Callable<T> task)
```

{% hint style="warning" %}
当使用 submit 方法时，若提交的任务抛出了异常，只能通过返回的 Future 获取异常。所以如果即使用 submit 方法，又没有保留返回的 Future，则必须保证提交的任务没有抛出异常，不然异常会被吃掉，不会打印出异常栈。

原因：使用 submit 后，会把任务包装成 FutureTask，在执行时会捕获异常，保存在 Future 里面，自然就不会再抛出异常了；而使用 execute 时，任务抛出的异常会直接再次抛出。

所以：**若不需要返回结果，建议使用 execute 方法**。
{% endhint %}

## Executors

Java 并发包提供了 Executors 可以快速创建线程池。

{% hint style="danger" %}
目前大厂的编码规范中基本上都**不建议使用 Executors** 了。因为 Executors 提供的很多方法默认使用的都是**无界**的 LinkedBlockingQueue。
{% endhint %}

## Future

`Future` 接口有如下方法：

```java
public interface Future<V> {
    // 取消任务
    boolean cancel(boolean mayInterruptIfRunning);
    // 任务是否已取消
    boolean isCancelled();
    // 任务是否已结束
    boolean isDone();
    // 任务执行结果
    V get() throws InterruptedException, ExecutionException;
    // 任务执行结果，支持超时
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

## FutureTask

`FutureTask`实现了`Runnable`和`Future` 接口。所以 `FutureTask` 可以很容易获取线程的执行结果。

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}

public class FutureTask<V> implements RunnableFuture<V> {
    public FutureTask(Callable<V> callable)
    public FutureTask(Runnable runnable, V result)
}
```

以华罗庚的烧水泡茶为例，可以有很多方法解决，比如 `Thread.join()`、`CountDownLatch` 等。用 `FutureTask` 来解决的方法如下：

```java
// 创建任务T2的FutureTask 
FutureTask<String> ft2 = new FutureTask<>(new T2Task()); 

// 创建任务T1的FutureTask 
FutureTask<String> ft1 = new FutureTask<>(new T1Task(ft2)); 

// 线程T1执⾏任务ft1 
Thread T1 = new Thread(ft1); 
T1.start(); 

// 线程T2执⾏任务ft2 
Thread T2 = new Thread(ft2); 
T2.start(); 

// 等待线程T1执⾏结果 
System.out.println(ft1.get());

// T1Task需要执⾏的任务： 洗⽔壶、烧开⽔、泡茶 
class T1Task implements Callable<String> { 
    FutureTask<String> ft2; 
    
    // T1任务需要T2任务的FutureTask 
    T1Task(FutureTask<String> ft2) { 
        this.ft2 = ft2; 
    } 
    
    @Override 
    String call() throws Exception {
        System.out.println("T1:洗⽔壶..."); 
        TimeUnit.SECONDS.sleep(1);
        
        System.out.println("T1:烧开⽔..."); 
        TimeUnit.SECONDS.sleep(15); 
        
        // 获取T2线程的茶叶 
        String tf = ft2.get(); 
        System.out.println("T1:拿到茶叶:"+tf);
        
        System.out.println("T1:泡茶..."); 
        return "上茶:" + tf;
    }
}

// T2Task需要执⾏的任务: 洗茶壶、洗茶杯、拿茶叶 
class T2Task implements Callable<String> {
    @Override
    String call() throws Exception {
        System.out.println("T2:洗茶壶...");
        TimeUnit.SECONDS.sleep(1);
        
        System.out.println("T2:洗茶杯..."); 
        TimeUnit.SECONDS.sleep(2);
        
        System.out.println("T2:拿茶叶..."); 
        TimeUnit.SECONDS.sleep(1); 
        
        return "⻰井";
    }
} 
```

![](../../.gitbook/assets/image%20%2818%29.png)

## CompletableFuture

### 概念

**异步化**：是利用多线程优化性能这个核心方案得以实施的基础。

Java 8 提供了 `CompletableFuture`来支持异步编程，Java 9 提供了更加完备的 Flow API，ReactiveX 的 Java 实现是 RxJava，使得在 Java 6就能使用异步编程。

重新实现一遍上一节的烧水泡茶：

![](../../.gitbook/assets/image%20%28211%29.png)

```java
//任务1：洗⽔壶->烧开⽔ 
CompletableFuture<Void> f1 = CompletableFuture.runAsync(()->{ 
	System.out.println("T1:洗⽔壶..."); 
	sleep(1, TimeUnit.SECONDS);

	System.out.println("T1:烧开⽔..."); 
	sleep(15, TimeUnit.SECONDS); 
}); 

//任务2：洗茶壶->洗茶杯->拿茶叶 
CompletableFuture<String> f2 = CompletableFuture.supplyAsync(()->{ 
	System.out.println("T2:洗茶壶..."); 
	sleep(1, TimeUnit.SECONDS);

	System.out.println("T2:洗茶杯..."); 
	sleep(2, TimeUnit.SECONDS);

	System.out.println("T2:拿茶叶..."); 
	sleep(1, TimeUnit.SECONDS); return "⻰井"; 
}); 

//任务3：任务1和任务2完成后执⾏：泡茶 
CompletableFuture<String> f3 = f1.thenCombine(f2, (__, tf)->{ 
	System.out.println("T1:拿到茶叶:" + tf); 
	System.out.println("T1:泡茶...");

	return "上茶:" + tf; 
}); 
```

### 创建 CompletableFuture

主要使用下面四种静态方法。

* 可以指定线程池，若不指定，则使用公用的`ForkJoinPool`线程池。
* ForkJoinPool 默认线程数为 CPU 核数，可以通过参数`-Djava.util.concurrent.ForkJoinPool.common.parallelism`设置。
* 若是 IO 较重的异步操作，建议使用自定义线程池。
* runAsync 使用 Runnable 接口，没有返回值。
* supplyAsync 使用 Supplier 接口，有返回值。

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
    public static CompletableFuture<Void> runAsync(Runnable runnable);
    public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor);
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier);
    public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor);
}
```

## CompletionStage

CompletableFuture 实现了 CompletionStage 接口。CompletionStage 定义了工作流的关系，工作流的关系主要有：**串行**关系、**并行**关系、**汇聚**关系。汇聚关系又分为：

* **AND 汇聚**：所有任务依赖都完成之后才开始执行当前任务。
* **OR 汇聚**：依赖的任务只要有一个完成就可以执行当前任务。

CompletionStage 除了定义工作流的关系之外，还要能够处理异常。

### 串行关系

* **thenApply（Function）**：既能接受参数，也支持返回值。
* **thenAccept（Consumer）**：能接受参数，不支持返回值。
* **thenRun（Runnable）**：不能接受参数，也不支持返回值。

```java
public <U> CompletionStage<U> thenApply(Function<? super T,? extends U> fn);
public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn);
public <U> CompletionStage<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor);

public CompletionStage<Void> thenAccept(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action, Executor executor);

public CompletionStage<Void> thenRun(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action);
public CompletionStage<Void> thenRunAsync(Runnable action, Executor executor);
```

### AND 汇聚关系

三者的差别与上文 **`Function`**、**`Consumer`**、**`Runnable`** 的差别一致。

```java
public <U,V> CompletionStage<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn);
public <U,V> CompletionStage<V> thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn, Executor executor);

public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action);
public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T, ? super U> action, Executor executor);

public CompletionStage<Void> runAfterBoth(CompletionStage<?> other, Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action);
public CompletionStage<Void> runAfterBothAsync(CompletionStage<?> other, Runnable action, Executor executor);
```

### OR 汇聚关系

三者的差别与上文 **`Function`**、**`Consumer`**、**`Runnable`** 的差别一致。

```java
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn);
public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T, U> fn, Executor executor);
              
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action);
public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action, Executor executor);

public CompletionStage<Void> runAfterEither(CompletionStage<?> other, Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action);
public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other, Runnable action, Executor executor);
```

### 异常处理

* **`exceptionally`**：相当于 `catch{}`代码块。
* **`whenComplete`**：相当于 `finally{}`代码块，无论异常是否发生都会执行，不支持返回结果。
* **`handleAsync`**：相当于`finally{}`代码块，支持返回结果，

```java
public CompletionStage<T> exceptionally(Function<Throwable, ? extends T> fn);

public CompletionStage<T> whenComplete(BiConsumer<? super T, ? super Throwable> action);
public CompletionStage<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action);
public CompletionStage<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action, Executor executor);

public <U> CompletionStage<U> handle(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn);
public <U> CompletionStage<U> handleAsync(BiFunction<? super T, Throwable, ? extends U> fn, Executor executor);
```

## CompletionService

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

