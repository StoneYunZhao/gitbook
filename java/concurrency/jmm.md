# JMM

## 前言

上一节讲到 Java 并发问题的源头可归结为**可见性**、**原子性**、**有序性**的问题。**Java 内存模型**可以解决**可见性**和**有序性**的问题。

Java 虚拟机规范中定义了 Java 内存模型（Java Memory Model，JMM），用于屏蔽掉各种硬件和操作系统的内存访问差异，以实现让 Java 程序在各种平台下都能达到**一致的并发效果**，JMM 规范了 Java 虚拟机与计算机内存是如何协同工作的：规定了一个线程如何和何时可以看到由其他线程修改过后的共享变量的值，以及在必须时如何同步的访问共享变量。

Java 内存模型在 Java1.5 时被重新修订。

CPU 缓存导致可见性，编译优化导致有序性，所以可以**按需禁用缓存和编译优化**来解决可见性和有序性问题。JMM 即规范了 JVM 如何提供按需禁用缓存和编译优化的方法。这些方法包括：

* volatile
* synchronized
* final
* Happens-Before 规则

## volatile

C 语言里面就有 volatile 关键字，在 C 里面的原始意义就是禁用 CPU 缓存，对 volatile 变量读写不能使用 CPU 缓存，必须从内存中读写。

如下代码，假设线程 A 执行 `writer()`，然后线程 B 执行 `reader()`，那么线程 B 看到 x 是多少呢？

* 若 JDK 低于1.5，则 x 等于0或42，这是因为 x 可能被 CPU 缓存导致可见性问题。
* 若 JDK 高于1.5，则 x 一定为42，这是因为 JMM 在 Java1.5 时被重新修订，即有了 Happens-Before 原则。

```java
class VolatileExample {
  int x = 0;
  volatile boolean v = false;
  public void writer() {
    x = 42;
    v = true;
  }
  public void reader() {
    if (v == true) {
      // 这里 x 会是多少呢？
    }
  }
}
```

## Happens-Before

Happens-Before 很多翻译都不对，它真正的意义是：**前一个操作的结果对后续操作是可见的**。

### 程序顺序性规则

一个线程中的每个操作，Happens-Before 于该线程中的任意后续操作。即：单线程中，程序前面对某个变量的修改一定是对后续操作可见的。

### volatile 变量规则

对一个`volatile`变量的写操作， Happens-Before 于后续对这个 `volatile` 变量的读操作。

注意，这个不像第一点，没有限定在单线程中。

### 传递性

如果 A Happens-Before B，且 B Happens-Before C，那么 A Happens-Before C。

{% hint style="warning" %}
现在我们使用前面三条规则来分析上面的代码，x=42 Happens-Before v=true， 线程 A 写 v Happens-Before 线程 B读 v，再根据传递性，所以线程 B 读到的 x 一定是 42
{% endhint %}

![](../../.gitbook/assets/image%20%2883%29.png)

### 监视器锁规则

对一个锁的**解锁** Happens-Before 于后续对这个锁的**加锁**。

### 线程`start()`规则

主线程 A 启动子线程 B 后，子线程 B 能够看到主线程在启动子线程 B 前的操作。

```java
Thread B = new Thread(()->{
  // 主线程调用 B.start() 之前
  // 所有对共享变量的修改，此处皆可见
  // 此例中，var==77
});
// 此处对共享变量 var 修改
var = 77;
// 主线程启动子线程
B.start();
```

### 线程`join()`规则

主线程 A 等待子线程 B 完成（主线程 A 通过调用子线程 B 的 join\(\) 方法实现），当子线程 B 完成后（主线程 A 中 `join()` 方法返回），主线程能够看到子线程的操作。

```java
Thread B = new Thread(()->{
  // 此处对共享变量 var 修改
  var = 66;
});
// 例如此处对共享变量修改，
// 则这个修改结果对线程 B 可见
// 主线程启动子线程
B.start();
B.join()
// 子线程所有对共享变量的修改
// 在主线程调用 B.join() 之后皆可见
// 此例中，var==66
```

## final

被`final`修饰的字段在构造器中一旦被初始化完成，并且构造器没有把`this`传递出去（this 引用逃逸是一件很危险的事情，其他线程有可能通过这个引用访问到“初始化了一半”的对象），那么在其他线程就能看见`final`字段的值（无须同步）。

```java
final int x;
// 错误的构造函数
public FinalFieldExample() { 
  x = 3;
  y = 4;
  // 此处就是讲 this 逸出，
  global.obj = this;
}
```

## 案例

下面代码运行后，会一直运行，CPU 一直使用率很高，8 个线程一直处于 while 循环，控制台没有任何输出。原因是 get 方法无法看见 set 方法的结果，不能从 Happens-Before 原则中导出 get 能看到 set 对 result 的修改。

```java
public class HappensBefore {
    private int result;

    private int getResult() {
        return result;
    }

    private synchronized void setResult(int result) {
        this.result = result;
    }

    public static void main(String[] args) throws InterruptedException {
        HappensBefore target = new HappensBefore();

        class Task implements Callable<Void> {
            @Override
            public Void call() {
                int x = 0;
                while (target.getResult() < 100) {
                    x++;
                }
                System.out.println(x);
                return null;
            }
        }

        ExecutorService threadPool = Executors.newFixedThreadPool(8);
        for (int i = 0; i < 8; i++) {
            threadPool.submit(new Task());
        }
        Thread.sleep(1000);
        target.setResult(200);
        threadPool.shutdown();
    }
}
```

需要解决上面问题，仅需要把 result 增加 volatile 修饰，甚至 set 方法的 synchronized 可以去掉。

```java
private volatile int result;
private void setResult(int result) {
    this.result = result;
}

// 输出
1259427188
1197924672
1236734699
1206846947
1204081779
1217966593
1200072101
1211415784
```

## Reference

* [https://zhuanlan.zhihu.com/p/29881777](https://zhuanlan.zhihu.com/p/29881777)
* [https://mp.weixin.qq.com/s/i9ES7u5MPWCv1n8jYU\_q\_w](https://mp.weixin.qq.com/s/i9ES7u5MPWCv1n8jYU_q_w)

