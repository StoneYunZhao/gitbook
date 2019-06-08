# Atomic Class

原子类是通过[无锁方案（CAS）](cas.md#qian-yan)实现的。

![](../../.gitbook/assets/image%20%2851%29.png)

## 基本数据类型

* AtomicBoolean
* AtomicInteger 
* AtomicLong

## 对象引用类型

* AtomicReference：会有 ABA 问题。
* AtomicStampedReference：无 ABA 问题，CAS 方法增加了版本号参数。
* AtomicMarkableReference：无 ABA 问题，将版本号简化成了一个 Boolean 值。

## 数组

可以原子化地更新数组里面的每一个元素。与基本数据类型的区别是：**每个方法多了一个数组的索引参数**。

* AtomicIntegerArray
* AtomicLongArray
* AtomicReferenceArray

## 对象属性更新器

可以原子化地更新对象的属性。与基本数据类型的区别是：**每个方法多了一个对象引用参数**。

* AtomicIntegerFieldUpdater
* AtomicLongFieldUpdater
* AtomicReferenceFieldUpdater

{% hint style="warning" %}
对象属性必须是 volatile 类型的，只有这样才能保证可见性。否则会抛出 IllegalArgumentException。
{% endhint %}

## 累加器

用来执行累加操作，相比原子化的基本数据类型，速度更快。但是不支持 compareAndSet\(\) 方法。

* DoubleAccumulator
* DoubleAdder
* LongAccumulator
* LongAdder

