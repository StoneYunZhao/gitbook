# Immutable

不可变类有很多好处，比如线程安全等。创建不可变类一般需要做如下处理：

* 将 class 声明为 final。
* 所有成员变量声明为 private final，并不要暴露 setter 方法。
* 对于 getter 或其它可能会返回内部状态的方法，使用 copy-on-write 的方式。
* 构造对象时，成员变量使用深度拷贝来初始化，而不是直接赋值。

## Integer

```java
public class Main {
    public static void main(String[] args) {

        Integer i = 1;
        Integer j = new Integer(1);

        Integer k = Integer.valueOf(1);

        System.out.println(i == j);
        System.out.println(j == k);
        System.out.println(i == k);


        Integer p = 234;
        Integer q = 234;
        System.out.println(p == q);

        Integer m = 3;
        Integer n = 3;
        System.out.println(m == n);
    }
}

```

```text
false
false
true
false
true
```

Integer部分源码：

```java
public final class Integer extends Number implements Comparable<Integer> {
    public Integer(int value) {
        this.value = value;
    }
    
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    
    private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
}
```

## String

```java
// Java 1.8 源码
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    private final char value[];
    
    // 不是final，第一调用 hashcode 的时候计算，
    // 所以在多线程时会重复计算，但是值一样。
    // 作者不缓存是因为性能问题，volatile 有性能开销，而考虑到重复计算的机会很小
    private int hash; 
    
    public String(String original) {
        this.value = original.value;
        this.hash = original.hash;
    }
}

// Java 1.11 源码
public final class String implements java.io.Serializable, Comparable<String>, CharSequence {
    private final byte[] value;
    private final byte coder;
    private int hash;
    
    static final boolean COMPACT_STRINGS;
    static {
        COMPACT_STRINGS = true;
    }
    
    public String(String original) {
        this.value = original.value;
        this.coder = original.coder;
        this.hash = original.hash;
    }
}
```

String 是不可变类，所以它原生保证了线程安全。也因为无法修改内部数据，可以看到**拷贝构造函数**不需要额外的复制数据。

Java 1.6 提供了 intern 方法，目的是把 String 缓存起来，起初缓存在方法区（PermGen）里面，由于容易导致 OOM，所以 1.7 移到了堆中，详见[方法区](../jvm/runtime-data-area.md#yong-jiu-dai-yu-yuan-kong-jian)。默认缓存大小也在不断扩大，最初是 1009，7u40被改为60013，可以使用`-XX:+PrintStringTableStatistics`打印，也可以使用`-XX:StringTableSize=N`修改。

intern 是一种显式排重，8u20 后推出了 G1 GC 下的字符串排重，通过将相同数据的字符串指向同一份数据，默认关闭的。需要使用 G1，并开启参数：`-XX:+UseStringDeduplication`。

### StringBuffer

由于 String 是不可变类，所以拼接、裁剪等操作都会产生新的 String 对象。StringBuffer 正是解决产生太多中间对象而设计的类。

* 本质是一个线程安全的可修改字符序列。
* 它通过给修改数据的方法都加上 synchronized 关键字来实现线程安全（这种方式可以用在我们平时开发线程安全的类，”过早优化是万恶之源“）。
* 除非有线程安全的需求，不然推荐使用 StringBuilder。

### StringBuilder

StringBuffer 的非线程安全版本。

### 小结

StringBuffer 和 StringBuilder 两者都继承了 AbstractStringBuilder，里面包含了基本操作，区别在于最终方法是否加了 synchronized。它们的底层数据结构（**包括 String**）为：

* Java 9 之前：char 数组。
* Java 9：byte 数组。

目前数组的初始容量是 16，扩容的方式是创建新的数组，抛弃旧的数组。所以若提前知道字符串长度，最好一开始就指定容量。

起初使用 char 数组，对于拉丁语系是一种浪费，Java 9 中引入了Compact Strings 的设计，改为了 byte 数组加上一个 coder 字段。



