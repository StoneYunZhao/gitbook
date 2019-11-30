# Synchronized & CAS

## 基础

造成线程安全问题的主要原因：

* 存在共享数据（也称临界资源）。
* 存在多条线程共同操作共享数据。

**解决方案**：当存在多个线程操作共享数据时，需要保证同一时刻有且只有一个线程在操作共享数据，其他线程必须等到该线程处理完数据后再进行，这种方式叫**互斥锁**。

Synchronized作用：

* 保证在同一个时刻，只有一个线程可以执行某个方法或者某个代码块。
* 保证一个线程的变化（主要是共享数据的变化）可被其他线程所看到（**可见性**，可替代`volatile`）。

Synchronized使用方式：

* 修饰实例方法，加锁对象为 **this**，方法修饰符 ****`ACC_SYNCHRONIZED`。
* 修饰静态方法，加锁对象为**类对象**。
* 修饰代码块，加锁对象需要**指定**，`monitorenter` 和 `monitorexit` 指令。

锁与资源：

* 锁和要保护的资源是有对应关系的，**受保护资源和锁之间的关系是N:1**。
* 若是没有关联的多个资源，尽量**用不同的锁**对受保护资源进行精细化管理，可以提升性能，即**细粒度锁**。
* 若多个资源有关联关系，应该用**同一把锁保护多个资源**。如两个账户转账，那么两个账户的余额是有关联关系的，那么应该用一把锁同时保护两个账户余额。

## Java 对象头

在 JVM，对象在内存中分为三块区域：

* **实例数据**：类的属性数据信息，包括父类的属性，此部分按照4字节对齐。
* 对齐填充：虚拟机要求对象的起始地址必须是8字节的整数倍。
* **对象头**：
  * **类型指针**：对象指向它的类对象的指针。
  * **Mark Word**：存储运行时数据，长度在32位和64位虚拟机分别占有32bit 和64bit（不考虑指针压缩）。由于运行时数据要记录很多数据，为了节约空间，采用动态的数据结构。

![Mark Word&#x533A;&#x57DF;&#x5728;&#x4E0D;&#x540C;&#x72B6;&#x6001;&#x65F6;&#x7684;&#x6570;&#x636E;](../../.gitbook/assets/image%20%28176%29.png)

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

依据：在大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得。

**获取锁**：

1. 检测 Mark Word 是否为可偏向状态，即是否为偏向锁1，锁标识位为01；
2. 若为可偏向状态，则测试线程 ID 是否为当前线程 ID，如果是，则执行步骤（5），否则执行步骤（3）；
3. 如果线程 ID 不为当前线程ID，则通过 CAS 操作竞争锁，竞争成功，则将Mark Word的线程ID 替换为当前线程 ID，否则执行线程（4）；
4. 通过CAS竞争锁失败，证明当前存在多线程竞争情况，当到达全局安全点，获得偏向锁的线程被挂起，偏向锁升级为轻量级锁，然后被阻塞在安全点的线程继续往下执行同步代码块；
5. 执行同步代码块。

**释放锁**：

采用了一种只有竞争才会释放锁的机制，线程是**不会主动**去释放偏向锁，需要等待其他线程来竞争。

1. 暂停拥有偏向锁的线程，判断锁对象石是否还处于被锁定状态；
2. 撤销偏向苏，恢复到无锁状态（01）或者轻量级锁的状态；

![](../../.gitbook/assets/image%20%28212%29.png)

### 轻量级锁

依据经验数据：对绝大部分的锁，在整个同步周期内都不存在竞争。

当关闭偏向锁或偏向锁升级，则会尝试获取轻量级锁。

**获取锁**：

1. 判断当前对象是否处于无锁状态（hashcode、0、01），若是，则 JVM 首先将在当前线程的栈帧中建立一个名为**锁记录（Lock Record）**的空间，用于存储锁对象目前的 Mark Word的拷贝（Displaced Mark Word）；否则执行步骤（3）；
2. JVM 利用 CAS 操作尝试将对象的 Mark Word 更新为指向 Lock Record 的指针，如果成功表示竞争到锁，则将锁标志位变成00（表示此对象处于**轻量级锁**状态），执行同步操作；如果失败则执行步骤（3）；
3. 判断当前对象的 Mark Word 是否指向当前线程的栈帧，如果是则表示当前线程已经持有当前对象的锁，则直接执行同步代码块；否则只能说明该锁对象已经被其他线程抢占了，这时轻量级锁需要膨胀为重量级锁，锁标志位变成10，后面等待的线程将会进入阻塞状态。

**释放锁**：

1. 取出在获取轻量级锁保存在 Displaced Mark Word 中的数据；
2. 用 CAS 操作将取出的数据替换当前对象的 Mark Word 中，如果成功，则说明释放锁成功，否则执行（3）；
3. 如果CAS操作替换失败，说明有其他线程尝试获取该锁，则需要在释放锁的同时需要唤醒被挂起的线程。

![](../../.gitbook/assets/image%20%2810%29.png)

### 自旋锁

基于在大多数情况下，线程持有锁的时间都不会太长，如果直接挂起操作系统层面的线程可能会得不偿失，毕竟操作系统实现线程之间的切换时需要从用户态转换到核心态，这个状态之间的转换需要相对比较长的时间。

通过执行一段无意义的循环（自旋）让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。

自旋不能替代阻塞，它虽然可以避免线程切换的开销，但是它占用了 CPU 的时间。默认自旋10次。

### 适应自旋锁

自旋锁默认10次，就算自旋了2次就可以获得锁，也会自旋10次。所以引入适应自旋锁。即自旋的次数不再是固定的，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

### 锁消除

Java虚拟机在JIT编译时，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁。

### 锁粗化

我们知道在使用同步锁的时候，需要让同步块的作用范围尽可能小，仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作数量尽可能缩小，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。

大多数情况没问题，但是如果一系列的加锁解锁导致性能损耗，可以将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁。

### 比较

![&#x91CD;&#x91CF;&#x7EA7;&#x9501;&#x3001;&#x8F7B;&#x91CF;&#x7EA7;&#x9501;&#x548C;&#x504F;&#x5411;&#x9501;&#x4E4B;&#x95F4;&#x8F6C;&#x6362;](../../.gitbook/assets/image%20%28197%29.png)

![](../../.gitbook/assets/image%20%2884%29.png)

## CAS

### 前言

可见性、有序性可以用 volatile 来解决，但是无法解决原子性问题。解决原子性问题一般有两种方式：

* **互斥锁**：synchronized、lock
* **无锁方案**：CAS + 自旋

无锁方案相对互斥锁方案，最大的好处就是性能**性能**。**互斥锁**需要加锁、解锁操作，这两个操作本身就消耗性能，若拿不到锁，还会进入阻塞状态，触发线程切换操作。**无锁方案**完全没有加锁、解锁的性能消耗。

### 实现原理

CAS（Compare and Swap），即比较与交换，是 CPU 提供的原子操作指令。CAS 指令包含三个参数：共享变量的内存地址 A、用于比较的值 B、共享变量的新值 C。并且只有当内存中地址 A 处的值等于 B 时，才能将内存中地址 A 处的值更新为新值 C。

Doug Lea 在 Java 同步框架中大量使用了 CAS。以 AtomicInteger 源码为例：

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
    
    public final int getAndAdd(int delta) {
        return unsafe.getAndAddInt(this, valueOffset, delta);
    }
}

// Unsafe的代码
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

* Unsafe，是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。
* 变量`valueOffset`，表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的。
* 变量`value`用`volatile`修饰，保证了多线程之间的内存可见性。

{% hint style="warning" %}
CAS 有 **ABA 问题**：如果变量 V 初次读取的时候是 A，并且在准备赋值的时候检查到它仍然是 A，那也不能说明在这期间没有被修改过，可能在这期间被修改成 B，然后又被修改成 A。有一个带有标记的原子引用类`AtomicStampedReference`，它可以通过控制变量值的版本来保证CAS的正确性。
{% endhint %}

