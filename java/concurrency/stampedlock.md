# StampedLock

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

