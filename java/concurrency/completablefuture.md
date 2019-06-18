# CompletableFuture

**异步化**：是利用多线程优化性能这个核心方案得以实施的基础。

Java 8 提供了 `CompletableFuture`来支持异步编程，Java 9 提供了更加完备的 Flow API，ReactiveX 的 Java 实现是 RxJava，使得在 Java 6就能使用异步编程。

## CompletableFuture

以上节[烧水泡茶](future.md#futuretask)为例：

![](../../.gitbook/assets/image%20%28151%29.png)

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

