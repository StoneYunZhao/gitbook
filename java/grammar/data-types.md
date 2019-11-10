# Data Types

Java 是一门强类型语言，意味着 Java 中的每个变量都申明了类型。Java 中有8种基本数据类型和引用类型。引用类型中 String 类和基本类型的包装类比较重要，它们都是[不可变类](../concurrency/concurrency-design-patterns/immutable.md)。

## 基本数据类型

| 类型 | 所占空间\(byte\) | 数据范围 | 默认值 | 包装类 |
| :---: | :---: | :---: | :---: | :---: |
| boolean | 1 | true和false | false | Boolean |
| byte | 1 | \[ - 2^7, 2^7 - 1 \] | 0 | Byte |
| char | 2 | Unicode \[0, 65535\] | u0000 | Character |
| short | 2 | \[ - 2^15, 2^15 - 1 \] | 0 | Short |
| int | 4 | \[ - 2^31, 2^31 - 1 \] | 0 | Integer |
| float | 4 | 32位 IEEE754 单精度 | 0.0f | Float |
| long | 8 | \[ - 2^63, 2^63 - 1 \] | 0L | Long |
| double | 8 | 64位 IEEE754 双精度 | 0.0 | Double |

* Java中的数值类型**都是有符号的**，不存在无符号的数值类型。
* Java中还存在一种基本类型void，有对应的包装类Void。

## 类型的自动转换

## 类型的强制转换

## 包装类

### Integer

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

Integer部分源码，借鉴了[享元模式](../../computer-science/design-patterns/flyweight.md)，但是没有照搬，内部维护了一个静态的对象池，仅缓存 \[-128, 127\] 之间的数字，在 JVM 启动时就创建好了：

这也是为什么 String 和 Integer 等不适合做锁，因为很容易使用公有的锁，导致锁竞争激烈

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

![](../../.gitbook/assets/image%20%2829%29.png)

* JDK 6 及以前：subString 方法会共享 char\[\]，所以会引起内存泄漏和内存溢出问题。
* JDK 7、JDK8：去掉两个字段，减少内存，subString 也不会共享 char\[\]。
* JDK9：更加节约内存。

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

String 是[不可变类](../concurrency/concurrency-design-patterns/immutable.md)，所以它原生保证了线程安全。也因为无法修改内部数据，可以看到**拷贝构造函数**不需要额外的复制数据。String 设计为不可变的好处：

* 安全性，不能被恶意篡改。
* hash 值不变，适合作为 HashMap 的 key。
* 可以实现字符串常量池。String 有两种创建方式：
  * 字符串常量的方式，String str = "abc"。JVM 会检查该对象是否在字符串常量池中，若存在则直接返回常量池中的引用；若不存在，则先在常量池中创建。
  * new 的方式，String str = new String\("abs"\)，在常量池中创建一个 String 对象，然后复制到堆内存中，返回堆内 String 对象的引用。

Java 1.6 提供了 intern 方法，会去字符串常量池中查看是否有该 String 对象，若有则返回常量池中的引用；若没有，则在常量池中创建，并返回常量池中的引用。目的是把 String 缓存起来，起初缓存在方法区（PermGen）里面，由于容易导致 OOM，所以 1.7 移到了堆中，详见[方法区](../jvm/runtime-data-area.md#yong-jiu-dai-yu-yuan-kong-jian)。默认缓存大小也在不断扩大，最初是 1009，7u40被改为60013，可以使用`-XX:+PrintStringTableStatistics`打印，也可以使用`-XX:StringTableSize=N`修改。

intern 是一种显式排重，也是[享元模式](../../computer-science/design-patterns/flyweight.md)的思想，8u20 后推出了 G1 GC 下的字符串排重，通过将相同数据的字符串指向同一份数据，默认关闭的。需要使用 G1，并开启参数：`-XX:+UseStringDeduplication`。

String 也使用了 [Copy-on-Write](../concurrency/concurrency-design-patterns/copy-on-write.md)（写时复制）的思想，比如 replace 方法：

```java
// Java 1.11 源码
public String replace(char oldChar, char newChar) {
    if (oldChar != newChar) {
        String ret = isLatin1() ? StringLatin1.replace(value, oldChar, newChar)
                                : StringUTF16.replace(value, oldChar, newChar);
        if (ret != null) {
            return ret;
        }
    }
    return this;
}

public static String replace(byte[] value, char oldChar, char newChar) {
    if (canEncode(oldChar)) {
        int len = value.length;
        int i = -1;
        while (++i < len) {
            if (value[i] == (byte)oldChar) {
                break;
            }
        }
        if (i < len) {
            if (canEncode(newChar)) {
                byte buf[] = new byte[len];
                for (int j = 0; j < i; j++) {    // TBD arraycopy?
                    buf[j] = value[j];
                }
                while (i < len) {
                    byte c = value[i];
                    buf[i] = (c == (byte)oldChar) ? (byte)newChar : c;
                    i++;
                }
                return new String(buf, LATIN1);
            } else {
                byte[] buf = StringUTF16.newBytesFor(len);
                // inflate from latin1 to UTF16
                inflate(value, 0, buf, 0, i);
                while (i < len) {
                    char c = (char)(value[i] & 0xff);
                    StringUTF16.putChar(buf, i, (c == oldChar) ? newChar : c);
                    i++;
                }
                return new String(buf, UTF16);
            }
        }
    }
    return null; // for string to return this;
}
```

关于 String 的使用有一些最佳实践，详见 [String 调优](../tuning/programming.md#string)。

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

