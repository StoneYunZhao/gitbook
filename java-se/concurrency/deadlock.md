# Deadlock

使用**细粒度锁**可以提高并行度，是性能优化的一个重要手段，但是细粒度锁可能导致**死锁**的问题。

**死锁**：一组互相竞争资源的线程因互相等待，导致“永久”阻塞的现象。

如下是一个转账的代码，仔细分析此代码，会发现会出现死锁的问题。假设线程 T1执行账户 A 向账户 B转账，线程 T2 执行账户 B 向账户 A 转账，那么可能发生死锁。

```java
class Account {
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    // 锁定转出账户
    synchronized(this) {              
      // 锁定转入账户
      synchronized(target) {           
        if (this.balance > amt) {
          this.balance -= amt;
          target.balance += amt;
        }
      }
    }
  } 
}
```

![](../../.gitbook/assets/image%20%2859%29.png)

Coffman 总结出，只有一下四个条件同时满足时，才能发生死锁：

1. **互斥**：共享资源 X 和 Y 只能被一个线程占用；
2. **占有且等待**：线程 T1 已经取得共享资源 X，在等待共享资源 Y 的时候，不释放共享资源 X；
3. **不可抢占**：其他线程不能强行抢占线程 T1 占有的资源；
4. **循环等待**：线程 T1 等待线程 T2 占有的资源，线程 T2 等待线程 T1 占有的资源，就是循环等待。

所以解决死锁的问题只需要破坏上面四条的其中一条。其中互斥没有破坏，还剩下三条。

### 破坏占有且等待

可以让线程 T1一次性获取共享资源 X、Y，即一次性申请所有资源。

```java
class Allocator {
  private List<Object> als = new ArrayList<>();
  // 一次性申请所有资源
  synchronized boolean apply(Object from, Object to){
    if(als.contains(from) || als.contains(to)){
      return false;  
    } else {
      als.add(from);
      als.add(to);  
    }
    return true;
  }
  // 归还资源
  synchronized void free(Object from, Object to){
    als.remove(from);
    als.remove(to);
  }
}
 
class Account {
  // actr 应该为单例
  private Allocator actr;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    // 一次性申请转出账户和转入账户，直到成功
    while(!actr.apply(this, target))；
    try{
      // 锁定转出账户，若仅有转账操作，则不需要加锁；加锁是为了锁定账户的其它操作。
      synchronized(this){              
        // 锁定转入账户，同上
        synchronized(target){           
          if (this.balance > amt){
            this.balance -= amt;
            target.balance += amt;
          }
        }
      }
    } finally {
      actr.free(this, target)
    }
  } 
}
```

这种方法正确性没有问题，但是效率在并发量大时会有问题，应该用**等待-通知**机制。

**等待 - 通知机制：**线程首先获取互斥锁，当线程要求的条件不满足时，释放互斥锁，进入等待状态；当要求的条件满足时，通知等待的线程，重新获取互斥锁。

```java
class Allocator {
  private List<Object> als;
  // 一次性申请所有资源
  synchronized void apply(Object from, Object to){
    // 经典写法
    while(als.contains(from) || als.contains(to)){
      try{
        wait();
      }catch(Exception e){}   
    } 
    als.add(from);
    als.add(to);  
  }
  // 归还资源
  synchronized void free(Object from, Object to){
    als.remove(from);
    als.remove(to);
    notifyAll();
  }
}
```

### 破坏不可抢占

这一点 synchronized 没法做到，但是 J.U.C 里面的 [Lock](lock-and-condition.md#lock) 可以轻松解决。

### 破坏循环等待

若对共享资源进行排序，按序申请资源，就不会存在循环等待的情况了。

```java
class Account {
  private int id;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    Account left = this        
    Account right = target;    
    if (this.id > target.id) { 
      left = target;           
      right = this;            
    }                          
    // 锁定序号小的账户
    synchronized(left){
      // 锁定序号大的账户
      synchronized(right){ 
        if (this.balance > amt){
          this.balance -= amt;
          target.balance += amt;
        }
      }
    }
  } 
}
```

