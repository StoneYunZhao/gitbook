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



## 生产者-消费者



