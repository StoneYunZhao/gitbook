# Lock & Condition

[序言](preface.md#java-yu-guan-cheng)里面已经提到，Java 使用 Lock 和 Condition 来实现管程，Lock 解决互斥问题，Condition 解决同步问题。

## Lock

### 为什么需要 Lock

在讲死锁时，提出[破坏不可抢占条件](deadlock.md#po-huai-bu-ke-qiang-zhan)方案，但这个方案 synchronized 没法解决。

synchronized 不能实现上述方案的**原因是**：synchronized 申请资源的时候，如果申请不到，线程直接进入阻塞状态了，而线程进入阻塞状态，啥都干不了，也释放不了线程已经占有的资源。

所以若要锁的实现能够解决上述问题，可以有如下方案：

* **能够响应中断**。如果阻塞状态的线程能够响应中断信号，也就是说当我们给阻塞的线程发送中断信号的时候，能够唤醒它，那它就有机会释放曾经持有的锁 A。
* **支持超时**。如果线程在一段时间之内没有获取到锁，不是进入阻塞状态，而是返回一个错误，那这个线程也有机会释放曾经持有的锁。
* **非阻塞地获取锁**。如果尝试获取锁失败，并不进入阻塞状态，而是直接返回，那这个线程也有机会释放曾经持有的锁。

这也是为什么 JDK 重新实现了套管程：

```java
public interface Lock {
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    boolean tryLock();
}
```

### Lock 怎么保证可见性

我们知道 Java 的 Happens-Before 原则[有一条关于 synchronized ](jmm.md#jian-shi-qi-suo-gui-ze)的，所以 synchronized 能够保证可见性。

但是 Lock 怎么保证可见性呢？Lock 内部持有一个 volatile 变量 state，获取和释放锁的时候都会读写 state，所以利用 [顺序性规则](jmm.md#cheng-xu-shun-xu-xing-gui-ze)、[volatile 变量规则](jmm.md#volatile-bian-liang-gui-ze)、[传递性](jmm.md#chuan-di-xing)，Lock 就可以保证可见性。

### 可重入锁

Lock 有一个实现类 `ReentrantLock`，即可重入锁：**线程可以重复获取同一把锁**。

### 公平锁与非公平锁

`ReentrantLock` 有一个有参构造函数，传入 fair 表示公平策略。

如果一个线程没有获得锁，就会进入等待队列，当有线程释放锁的时候，就需要从等待队列中唤醒一个等待的线程。**公平锁**：谁等待的时间长，就唤醒谁；**非公平锁**：有可能等待时间短的线程反而先被唤醒。

### 用锁的最佳实践

1. 永远只在更新对象的**成员变量**时加锁
2. 永远只在访问**可变的**成员变量时加锁
3. 永远不在调用其他对象的方法时加锁

## Condition

