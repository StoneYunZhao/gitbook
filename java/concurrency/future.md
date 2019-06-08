# Future & FutureTask

## submit

`ThreadPoolExecutor` 的 `execute()` 方法不能取得任务结果，但是 `submit()` 方法可以。

```java
// Runnable 的 run 方法没有返回值，所以这个方法仅能用来判断任务是否结束
Future<?> submit(Runnable task)

// 假设这个方法返回 f，f.get() 的返回值就是传给 submit() 方法的参数 result。
<T> Future<T> submit(Runnable task, T result)

<T> Future<T> submit(Callable<T> task)
```

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

![](../../.gitbook/assets/image%20%2841%29.png)

