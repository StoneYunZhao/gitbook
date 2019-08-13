# Division

并发编程领域的核心问题可分为：[分工、同步、互斥](../preface.md#fen-gong-tong-bu-hu-chi)。同步和互斥问题多源自微观，而分工往往来自宏观。

解决分工问题也有一系列的设计模式，常用的有 Thread-Per-Message、Work Thread、生产者-消费者。

## Thread-Per-Message

简单来说，就是为每个任务分配一个独立的线程去处理。

比如一个 Http Server，在主线程中只接受请求，处理请求通过创建一个子线程处理，不然整个服务同一时间只能处理一个请求。

### Thread 实现

Thread-Per-Message 在网络编程的服务端使用非常广泛，最简单的服务端就是 echo 服务，即客户端发送什么，服务端就响应相同的内容。

```java
public static void main(String[] args) throws IOException {
    try (ServerSocketChannel ssc = ServerSocketChannel.open().bind(new InetSocketAddress(8080))) {
        try {
            while (true) {
                SocketChannel sc = ssc.accept();
                new Thread(() -> {
                    try {
                        ByteBuffer rb = ByteBuffer.allocate(2048);
                        sc.read(rb);
                        rb.flip();
                        sc.write(rb);
                        sc.close();
                    } catch (IOException e) {
                        throw new UncheckedIOException(e);
                    }
                }).start();
            }
        } finally {
            ssc.close();
        }
    }
}
```

### Fiber 实现

上面 Thread 实现有一个非常严重的问题，原因是 Java 的线程是很重量的，Java 的线程对应操作系统的线程，优点是技术很成熟，缺点是创建成本高。

所以 Thread-Per-Message 这种设计模式对 Java 语言确实不友好，但是其它语言就不一样了，比如 Go、Lua 有轻量级线程的概念，Kotlin 也有 coroutine。

Java 也有一个 Loom 项目，解决轻量级线程的问题，在 Loom 项目中，轻量级线程叫做 Fiber。

```java
public static void main(String[] args) throws IOException {
    try (ServerSocketChannel ssc = ServerSocketChannel.open().bind(new InetSocketAddress(8080))) {
        try {
            while (true) {
                SocketChannel sc = ssc.accept();
                Fiber.schedule(() -> {
                    try {
                        ByteBuffer rb = ByteBuffer.allocate(2048);
                        sc.read(rb);
                        rb.flip();
                        sc.write(rb);
                        sc.close();
                    } catch (IOException e) {
                        throw new UncheckedIOException(e);
                    }
                });
            }
        } finally {
            ssc.close();
        }
    }
}
```

Java 还有一个开源的协程库叫 [Quasar](https://github.com/puniverse/quasar)。

## Work Thread

上一节的 Thread-Per-Message 模式不适合高并发的场景，原因在于频繁的创建、销毁现场非常消耗性能。可以使用 **Work-Thread** 模式来避免上述问题，原理就是用阻塞队列做任务池，创建固定数量的线程来消费任务。可以看出，这就是常见的**线程池**的模式。

```java
ExecutorService es = Executors.newFixedThreadPool(500);
try (ServerSocketChannel ssc = ServerSocketChannel.open().bind(new InetSocketAddress(8080))) {
    try {
        while (true) {
            SocketChannel sc = ssc.accept();
            es.execute(() -> {
                try {
                    ByteBuffer rb = ByteBuffer.allocate(2048);
                    sc.read(rb);
                    rb.flip();
                    sc.write(rb);
                    sc.close();
                } catch (IOException e) {
                    throw new UncheckedIOException(e);
                }
            });
        }
    } finally {
        ssc.close();
        es.shutdown();
    }
}
```

### 创建线程池的建议

* 使用**有界队列**来接受任务。
* 清晰地指明拒绝策略。
* 给线程赋予一个业务相关的名字。

### 避免线程死锁

线程池有一种线程死锁的场景：如果提交到线程池的任务有依赖关系，那么有可能导致线程死锁。如下例子：

```java
ExecutorService es = Executors.newSingleThreadExecutor();
es.execute(() -> {
    try {
        String qq = es.submit(() -> "QQ").get();
        System.out.println(qq);
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
});
```

下面例子，线程全部阻塞在 l2.await\(\)：

```java
ExecutorService es = Executors.newFixedThreadPool(2);
CountDownLatch l1 = new CountDownLatch(2);
for (int i = 0; i < 2; i++) {
    es.execute(() -> {
        CountDownLatch l2 = new CountDownLatch(2);
        for (int j = 0; j < 2; j++) {
            es.execute(l2::countDown);
        }
        try {
            l2.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        l1.countDown();
    });
}
l1.await();
es.shutdown();
```

解决办法：

1. 增大线程池数量，但是无法确定任务数量或任务数量很多，此方法不可用。
2. 为不同的任务创建各自的线程池。

{% hint style="warning" %}
提交到线程池的任务一定要相互独立！
{% endhint %}

## 生产者-消费者

**原理**：一组**生产者线程**负责生产任务，把任务放入**任务队列**，一组**消费者线程**从任务队列中获取任务执行。

Java 的线程池即是生产者-消费者模式。

优点：

* **解耦**：生产者和消费者没有依赖关系。
* **异步**：生产者无需等待任务执行完成。
* **平衡生产者和消费者的速度**：如果生产者和消费者的速度是 1:3，则消费者只需要 1/3 的线程数量，减少开销。
* **支持批量执行**：比如生产者生产的每个任务都是往数据库中插入一条数据，那么消费者可以一次性从任务队列中取出多个任务，然后组合成一个批量插入的 SQL 语句，这样就大量减少了对数据库的写入操作。

```java
private BlockingQueue<Task> bq = new LinkedBlockingQueue<>(2000);

public void start() {
    ExecutorService es = Executors.newFixedThreadPool(5);
    for (int i = 0; i < 5; i++) {
        es.execute(() -> {
            while (true) {
                try {
                    List<Task> tasks = pollTasks();
                    // 执行任务
                } catch (InterruptedException e) { }
            }
        });
    }
}

private List<Task> pollTasks() throws InterruptedException {
    List<Task> ts = new LinkedList<>();
    Task t = bq.take();
    while (t != null) {
        ts.add(t);
        t = bq.poll();
    }
    return ts;
}
```

