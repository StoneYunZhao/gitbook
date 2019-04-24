# CAS

可见性、有序性可以用 volatile 来解决，但是无法解决原子性问题。解决原子性问题一般有两种方式：

* **互斥锁**：synchronized、lock
* **无锁方案**：CAS + 自旋

无锁方案相对互斥锁方案，最大的好处就是性能**性能**。**互斥锁**需要加锁、解锁操作，这两个操作本身就消耗性能，若拿不到锁，还会进入阻塞状态，触发线程切换操作。**无锁方案**完全没有加锁、解锁的性能消耗。

## 实现原理

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

## 原子类

![](../../.gitbook/assets/image%20%2823%29.png)

### 基本数据类型

* AtomicBoolean
* AtomicInteger 
* AtomicLong

### 对象引用类型

* AtomicReference：会有 ABA 问题。
* AtomicStampedReference：无 ABA 问题，CAS 方法增加了版本号参数。
* AtomicMarkableReference：无 ABA 问题，将版本号简化成了一个 Boolean 值。

### 数组

可以原子化地更新数组里面的每一个元素。与基本数据类型的区别是：**每个方法多了一个数组的索引参数**。

* AtomicIntegerArray
* AtomicLongArray
* AtomicReferenceArray

### 对象属性更新器

可以原子化地更新对象的属性。与基本数据类型的区别是：**每个方法多了一个对象引用参数**。

* AtomicIntegerFieldUpdater
* AtomicLongFieldUpdater
* AtomicReferenceFieldUpdater

{% hint style="warning" %}
对象属性必须是 volatile 类型的，只有这样才能保证可见性。否则会抛出 IllegalArgumentException。
{% endhint %}

### 累加器

用来执行累加操作，相比原子化的基本数据类型，速度更快。但是不支持 compareAndSet\(\) 方法。

* DoubleAccumulator
* DoubleAdder
* LongAccumulator
* LongAdder

