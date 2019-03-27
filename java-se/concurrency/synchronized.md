# Synchronized

## 基础

造成线程安全问题的主要原因：

* 存在共享数据（也称临界资源）。
* 存在多条线程共同操作共享数据。

**解决方案**：当存在多个线程操作共享数据时，需要保证同一时刻有且只有一个线程在操作共享数据，其他线程必须等到该线程处理完数据后再进行，这种方式叫**互斥锁**。

Synchronized作用：

* 保证在同一个时刻，只有一个线程可以执行某个方法或者某个代码块。
* 保证一个线程的变化（主要是共享数据的变化）可被其他线程所看到（**可见性**，可替代`volatile`）。

Synchronized使用方式：

* 修饰实例方法，加锁对象为 **this**。
* 修饰静态方法，加锁对象为**类对象**。
* 修饰代码块，加锁对象需要**指定**。

## Java 对象头

在 JVM，对象在内存中分为三块区域：

* **实例数据**：类的属性数据信息，包括父类的属性，此部分按照4字节对齐。
* 对齐填充：虚拟机要求对象的起始地址必须是8字节的整数倍。
* **对象头**：
  * **类型指针**：对象指向它的类对象的指针。
  * **Mark Word**：存储运行时数据，长度在32位和64位虚拟机分别占有32bit 和64bit（不考虑指针压缩）。由于运行时数据要记录很多数据，为了节约空间，采用动态的数据结构。

![Mark Word&#x533A;&#x57DF;&#x5728;&#x4E0D;&#x540C;&#x72B6;&#x6001;&#x65F6;&#x7684;&#x6570;&#x636E;](../../.gitbook/assets/image%20%2831%29.png)

## Monitor

当锁标志位为 10 时，指针指向的是一个 monitor 对象。monitor 对象可以与对象一起创建于销毁，也可以在某个线程试图获取对象锁的时候生成，实现方式由虚拟机实现。

HotSpot 虚拟机由 [ObjectMonitor](https://github.com/JetBrains/jdk8u_hotspot/blob/master/src/share/vm/runtime/objectMonitor.hpp) 实现，主要数据结构如下：

```cpp
ObjectMonitor() {
    _header       = NULL;
    _count        = 0; //记录个数
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  }
```

* **\_EntrySet**：当多个线程同时获取某一对象的锁时，会先进入\_EntrySet集合。每个线程被封装成 ObjectWaiter 对象。
* **\_owner**：当某个线程获取到对象的锁时，会把 \_Owner 变量设置为当前线程。
* **\_count**：同时 count 加1。
* **\_WaitSet**：若线程调用 wait\(\) 方法，则会进入 WaitSet 集合，并释放锁（即 owner字段置为 NULL，count 减 1）。
* 若线程执行完毕，也会释放锁。

## 锁优化

锁的状态总共有四种：**无锁状态**、**偏向锁**、**轻量级锁**和**重量级锁**。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁（锁的升级是**单向**的）。

### 偏向锁

在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得。

### 轻量级锁

经验数据：对绝大部分的锁，在整个同步周期内都不存在竞争。

### 自旋锁

基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间。

### 锁消除

Java虚拟机在JIT编译时，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁。

### 比较

![&#x91CD;&#x91CF;&#x7EA7;&#x9501;&#x3001;&#x8F7B;&#x91CF;&#x7EA7;&#x9501;&#x548C;&#x504F;&#x5411;&#x9501;&#x4E4B;&#x95F4;&#x8F6C;&#x6362;](../../.gitbook/assets/image%20%2834%29.png)

![](../../.gitbook/assets/image%20%2813%29.png)

## 参考

[https://blog.csdn.net/zhoufanyang\_china/article/details/54601311](https://blog.csdn.net/zhoufanyang_china/article/details/54601311)  
[https://blog.csdn.net/u012465296/article/details/53022317](https://blog.csdn.net/u012465296/article/details/53022317)  
[https://blog.csdn.net/javazejian/article/details/72828483](https://blog.csdn.net/javazejian/article/details/72828483)  
[https://zhuanlan.zhihu.com/p/29866981](https://zhuanlan.zhihu.com/p/29866981)

