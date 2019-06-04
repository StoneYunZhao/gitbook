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

### ReentrantLock

Lock 有一个实现类 `ReentrantLock`，即可重入锁：**线程可以重复获取同一把锁**。

底层基于 AQS，每当同一个线程获得一次这把锁，state 变量加1，释放一次 state 变量减1，最终当线程不持有锁时，state 为0。

### 公平锁与非公平锁

`ReentrantLock` 有一个有参构造函数，传入 fair 表示公平策略。

如果一个线程没有获得锁，就会进入等待队列，当有线程释放锁的时候，就需要从等待队列中唤醒一个等待的线程。**公平锁**：谁等待的时间长，就唤醒谁；**非公平锁**：有可能等待时间短的线程反而先被唤醒。

### 用锁的最佳实践

1. 永远只在更新对象的**成员变量**时加锁
2. 永远只在访问**可变的**成员变量时加锁
3. 永远不在调用其他对象的方法时加锁

## Condition

Condition 实现了管程模型里面的条件变量。一个 Lock 可以有多个条件变量，这个是与 synchronized 一个很大的区别。

#### 利用两个条件变量实现阻塞队列

```java
public class BlockedQueue<T>{
  final Lock lock = new ReentrantLock();
  // 条件变量：队列不满  
  final Condition notFull = lock.newCondition();
  // 条件变量：队列不空  
  final Condition notEmpty = lock.newCondition();
 
  // 入队
  void enq(T x) {
    lock.lock();
    try {
      while (队列已满){
        // 等待队列不满
        notFull.await();
      }  
      // 省略入队操作...
      // 入队后, 通知可出队
      notEmpty.signal();
    }finally {
      lock.unlock();
    }
  }
  // 出队
  void deq(){
    lock.lock();
    try {
      while (队列已空){
        // 等待队列不空
        notEmpty.await();
      }  
      // 省略出队操作...
      // 出队后，通知可入队
      notFull.signal();
    }finally {
      lock.unlock();
    }  
  }
}
```

由此可见：

* `Condition.await()` 对应 `Object.wait()`
* `Condition.signal()` 对应 `Object.notify()`
* `Condition.signalAll()` 对应 `Object.notifyAll()`

## ReadWriteLock

在生产中经常有读多写少的场景，比如缓存。针对这种情况，Java 提供了一个接口`ReadWriteLock`。

* 允许多个线程同时读共享变量。
* 只允许一个线程写共享变量。
* 当一个线程在写共享变量时，不允许其它线程读和写操作。

### 缓存实现

```java
class Cache<K,V> {
  final Map<K, V> m = new HashMap<>();
  final ReadWriteLock rwl = new ReentrantReadWriteLock();
  // 读锁
  final Lock r = rwl.readLock();
  // 写锁
  final Lock w = rwl.writeLock();
  // 读缓存
  V get(K key) {
    r.lock();
    try { return m.get(key); }
    finally { r.unlock(); }
  }
  // 写缓存
  V put(String key, Data v) {
    w.lock();
    try { return m.put(key, v); }
    finally { w.unlock(); }
  }
}
```

### 缓存数据按需加载

缓存中的数据是需要初始化的，若数据量小，可以直接一次性全部加载；若数据量大，则要按需加载（懒加载），即当查询的时候再加载。

```java
  V get(K key) {
    V v = null;
    // 读缓存
    r.lock();         
    try {
      v = m.get(key); 
    } finally{
      r.unlock();     
    }
    // 缓存中存在，返回
    if(v != null) {   
      return v;
    }  
    // 缓存中不存在，查询数据库
    w.lock();         
    try {
      // 再次验证
      // 其他线程可能已经查询过数据库
      v = m.get(key); 
      if(v == null){  
        // 查询数据库
        v= 省略代码无数
        m.put(key, v);
      }
    } finally{
      w.unlock();
    }
    return v; 
  }
```

### 读写锁升级与降级

把上面的代码改成如下是否可行？

```java
// 读缓存
r.lock();         
try {
  v = m.get(key); 
  if (v == null) {
    w.lock();
    try {
      // 再次验证并更新缓存
      // 省略详细代码
    } finally{
      w.unlock();
    }
  }
} finally{
  r.unlock();     
}
```

答案是不行，会导致写锁永远等待。

* **锁的升级**：先获取了读锁，再获取写锁。这是**不允许**的。
* **锁的降级**：先获取了写锁，再获取读锁。这是**允许**的。

### 总结

* 读写锁也支持公平和非公平模式。
* 只有**写锁支持条件变量**，读锁不支持。

## StampedLock

Java 1.8 提供，性能比 ReadWriteLock 更好。支持三种模式：

* **写锁**：与 ReadWriteLock的写锁类似，只允许一个线程获取。获取时返回一个 stamp，解锁时需要传入此 stamp。
* **悲观读锁**：与 ReadWriteLock 读锁类似，允许多个线程同时获取，与写锁互斥。获取时返回一个 stamp，解锁时需要传入此 stamp。
* **乐观读**：注意没加锁字，因为乐观读是**无锁**的。允许一个线程获取写锁。

```java
class Point {
  private int x, y;
  final StampedLock sl = new StampedLock();
  // 计算到原点的距离  
  int distanceFromOrigin() {
    // 乐观读
    long stamp = sl.tryOptimisticRead();
    // 读入局部变量，
    // 读的过程数据可能被修改
    int curX = x, curY = y;
    // 判断执行读操作期间，
    // 是否存在写操作，如果存在，
    // 则 sl.validate 返回 false
    if (!sl.validate(stamp)){
      // 升级为悲观读锁
      stamp = sl.readLock();
      try {
        curX = x;
        curY = y;
      } finally {
        // 释放悲观读锁
        sl.unlockRead(stamp);
      }
    }
    return Math.sqrt(
      curX * curX + curY * curY);
  }
}
```

上述代码首先乐观读，但是 x、y 可能被修改，因此需要验证一下，若验证不通过，升级为读锁。若不升级为读锁，需要在循环里面反复乐观读，浪费 CPU，所以**最佳实践是升级为读锁**。

### 数据库的乐观锁

* 先从数据库读一条带 version 字段的记录。
* 然后在程序中对数据做业务修改。
* 最后写入数据库时采用 `update set version = version +  where version = XX and id = XX`的语句，若返回1，则说明更新成功，期间没有人修改这条数据；若返回0，则更新失败，说明有人修改过此数据。

可见，数据库乐观锁的 version 字段与 StampedLock 的 stamp 是同样的意义。

### 结论

StampedLock 的功能是 ReadWriteLock 的**子集**。

* StampedLock 不支持可重入；
* StampedLock 的悲观读锁和写锁都不支持条件变量；
* 若线程阻塞在 StampedLock 的 readLock\(\) 或者 writeLock\(\) 上时，一定**不要调用中断**操作；如果需要支持中断功能，一定使用可中断的悲观读锁 `readLockInterruptibly()` 和写锁 `writeLockInterruptibly()`。
* StampedLock **支持锁降级**`tryConvertToReadLock()`与**升级**`tryConvertToWriteLock()`。

StampedLock 读模板：

```java
final StampedLock sl = new StampedLock();
 
// 乐观读
long stamp = sl.tryOptimisticRead();
// 读入方法局部变量
......
// 校验 stamp
if (!sl.validate(stamp)){
  // 升级为悲观读锁
  stamp = sl.readLock();
  try {
    // 读入方法局部变量
    .....
  } finally {
    // 释放悲观读锁
    sl.unlockRead(stamp);
  }
}
// 使用方法局部变量执行业务操作
......
```

StampedLock 写模板：

```java
long stamp = sl.writeLock();
try {
  // 写共享变量
  ......
} finally {
  sl.unlockWrite(stamp);
}
```

