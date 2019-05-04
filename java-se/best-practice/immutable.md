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

