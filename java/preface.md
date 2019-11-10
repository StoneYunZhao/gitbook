# Preface

## Java 平台

Java 是一门面向对象的语言，有两个显著的特征：

* 书写一次，到处运行（Write once，run anywhere）
* 垃圾收集（GC，Garbage Collection）

![Java &#x5E73;&#x53F0;&#x6982;&#x89C8;](../.gitbook/assets/image%20%2893%29.png)

Java 分为**编译期**和**运行期**，这里的编译与 C/C++ 编译不同，Java 编译成字节码，而 C/C++ 编译成机器码。

运行期 Java 加载字节码，**解释**或**编译（注意：这里的编译又与上面说的编译不一样）**执行，如 JDK 8 是解释和编译混合执行。Server 模式的 JVM 会进行上万次调用收集足够信息来高效编译，而 client 模式是1500 次。HotSpot 内置了两个 JIT（Just-In-Time），C1 对应 client 模式，适合启动速度高的应用，如桌面应用；C2 对应 server 模式，适合长时间运行的应用，如服务端应用，采用**分层编译**（TieredComplilation）。

* `-Xint`：仅解释执行。
* `-Xcomp`：不要进行解释执行，会导致启动慢很多，而且不能根据运行情况有效优化，如分支预测。

**AOT**（Ahead-of-Time Compilation）：直接将字节码编译成机器码，如 JDK 9 引入了 AOT，增加了 jaotc 工具。**AOT 可以和分层编译结合**，不是二选一。

## JDK 1.7 新特性

### Switch 可以接受 String类型

从本质上来讲，Switch 对 String 的支持，其实是对 int 类型的匹配。调用 String 的 `hashCode()` 方法，得到一个 int 值，若匹配成功则接着调用 `equals()` 方法。所以 String 变量不能为 `null`，case 子句中的 String 也不能为 `null`。

### Catch捕获多个异常

可以在 `catch` 代码块中捕获多个异常 `catch (NumberFormatException | NullPointerException e)`。

### 二进制字面量

byte、short、int、long 可以用二进制表示。`0b001, 0B111`。

### 数字分隔符

数字中间可以增加分隔符，在**编译**的时候**下划线**会被去掉。`int c = 123_456`。

### 泛型类型推断

Java7以前：`Map<String, String> dict = new HashMap<String, String>()`

Java7：`Map<String, String> dict = new HashMap<>()`。

### Try-with-resources

```text
try (InputStream in = new FileInputStream("1.txt")) {
    
} catch (IOException e) {
    e.printStackTrace();
}
```

### fork/join

增加了 fork/join 框架。

## JDK 1.8 新特性

### Lambda 表达式

Lambda 表达式是一个匿名函数。基本语法：

`(parameters) -> expression 或 (parameters) -> {statements;}`。

通过函数式接口来实现的，Java8 新增了特殊的注解`@FunctionalInterface`。

### 方法引用

可以直接饮用 Java 类或对象的方法，可以看做是更加简洁的 Lambda 表达式。

* 构造方法：`ClassName::new`。
* 静态方法：`ClassName::method`。

### Date-time Package

新增了 LocalDate 和 LocalTime 接口，把月份和星期都改成了 enum，把上述两个类变成不可变类，所以线程安全了。

### 接口默认实现和静态方法

```text
public interface Inter {
    void f();
    
    default void g(){
        System.out.println(1);
    }

    static void h() {
        System.out.println(2);
    }
}
```

### HashMap 性能

当 Hash 冲突时，Java8 以前用链表存储，Java8 开始，当冲突节点个数 `>=TREEIFY_THRESHOLD - 1`时，将采用红黑树存储。

### 注解

JDK1.5 引入了注解机制，但是相同的注解在同一位置只能声明一次。

重复注解：JDK1.8 可以使用相同的注解在同一地方声明多次。

应用范围扩展：可以给局部变量、泛型、方法异常等提供注解。

### 参数名称

在编译的时候增加 -parameters 选项，反射 `Parameter.getName()`可以获取参数名。

### Optional

处理空指针异常，增强了代码的可读性。

### Stream

函数式编程风格。

### Base64

把 Base64 编码添加到了标准类库。

