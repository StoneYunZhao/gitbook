# Exceptions

Java 语言提供了完善的异常处理机制。

![](../../.gitbook/assets/image%20%28203%29.png)

Java 只有 Throwable 才能被抛出（throw）和和捕获（catch）。

* **Exception**：程序正常运行中，可以预料的意外情况，应该被捕获并进行相应的处理。
  * **受检异常**：必须显示的捕获处理，编译期的一部分。
  * **运行时异常（RuntimeException）**：不必在编译期强制检查。
* **Error**：大部分会导致程序处于非正常、不可恢复的状态。

### NoClassDefError & ClassNotFoundException

* 一个是**错误**，一个是**异常**。我们应该从异常中恢复程序，而不是从错误中。
* **ClassNotFoundException**：`Class.forName` 可能抛出。
* **NoClassDefFoundError**：ClassLoader 尝试加载类时找不到类的定义，在编译时是存在的，运行时却找不到了。可能是打包时漏掉了某些类、jar 包篡改等。

### try-with-resources

由于异常处理比较繁琐，有很多千篇一律的代码，所以 Java 语言引入了一些便利的语法（语法糖）。利用 try-with-resources 语法可以自动 close 那些扩展了AutoCloseable 或 Closeable 对象：

```java
try (BufferedReader br = new BufferedReader(…);
     BufferedWriter writer = new BufferedWriter(…)) {
}
```

### multiple catch

multiple catch 也是 Java 新出现的一种便利的语法：

```java
try {
}
catch ( IOException | XEception e) {
} 

```

###  Best practices

对于异常的处理有一些前人总结的最佳实践：

* **不要捕获类似 Exception 这样的通用异常**。
* **不要生吞（swallow）异常**。
* **Throw early，catch late**：即第一时间抛出异常。

### 自定义异常

在业务系统中，我们经常需要自定义异常。一般需要考虑如下两点：

* 是否需要定义成受检异常。
* 在保证诊断信息足够的同时，也要考虑**避免敏感信息**，比如IP、密码等。

### 问题

业界有很多人争论说 Java 的**受检异常是一个设计错误**，他们的理由如下：

* 受检异常设计的目的是：我们捕获了异常，然后恢复程序。但是大多数情况不能恢复。
* 不兼容 Functional Program。

不过也有反对者，他们的理由是确认有一个异常被捕获后可以被恢复，比如 IO、网络等。

从性能的角度，Java 的异常会有如下问题：

* try-catch 会产生额外的性能开销。
* 每实例化一个 Exception 对象，都会对当时的栈进行快照，这个操作比较重。

