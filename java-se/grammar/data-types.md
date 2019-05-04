# Data Types

Java是一门强类型语言，意味着Java中的每个变量都申明了类型。Java中有8种基本数据类型和引用类型。

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

## 不可变类

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

### String

