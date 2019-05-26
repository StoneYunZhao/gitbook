# Programming

## String

String 对象是 Java 语言中最重要的数据类型，往往在内存中占用最大。合理地使用 String，可以提升系统的整体性能。请先了解 [String 类型](../grammar/data-types.md#string)。

### 显式使用 StringBuilder

```java
String s1 = "ab" + "cd" + "ef";

// 字节码
// LDC "abcdef"
```

```java
String s2 = "abc";
for(int i=0; i<1000; i++) {
    s2 = s2 + i;
}

// 反编译
String str = "abc";
for(int i=0; i<1000; i++) {
    str = (new StringBuilder(String.valueOf(str))).append(i).toString();
}

```

如上面代码，虽然编译器会优化，但是每次循环都创建一个 StringBuilder 对象，会降低性能，所以建议显式使用 StringBuilder 对象。

### String.intern\(\)

调用 intern\(\) 方法后，堆内原有的 String 对象就没有引用了，可以被 GC。

```java
String str1= "abc";
String str2= new String("abc");
String str3= str2.intern();
assertSame(str1==str2);
assertSame(str2==str3);
assertSame(str1==str3)

// false false true
```

要注意过度使用 intern 也会使常量池太大，遍历的时间复杂度增加。所以要根据需求权衡。

### String.split

split 使用正则表达式，但正则的性能不稳定，使用不恰当会引起回溯问题，导致 CPU 居高不下。

若 indexOf 方法能够满足需求，尽量使用它。

{% hint style="info" %}
split 有两种情况不会使用正则：

* 传入参数长度为 1，且不包含 .$\|\(\)\[{^?\*+\\；
* 传入参数长度为 2，第一个字符是 \，且第二个字符不是 ASCII 数字或字母。
{% endhint %}

