# CAS

CAS（Compare and Swap），即比较与交换。Doug Lea 在 Java 同步框架中大量使用了 CAS。

以 AtomicInteger 源码为例：

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

