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

## 正则表达式

正则表达式使用单个字符串来描述、匹配一系列符合某个句法规则的字符串，很多语言都实现了它。

正则表达式由四种元素组成，详见 [PCRE 表达式全集](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)：

* 普通字符：字母\[a-zA-Z\]、数字\[0-9\]、下划线\[\_\]、标点等
* 标准字符：能够与多种普通字符匹配的简单表达式，如 \d（数字）、\w（包括下划线的单词字符）、\s（空白字符）等。
* 限定字符：用于表示数量，\*、+、?、{n} 等。
* 定位字符：符合某种条件的位置，$、^ 等。

### 引擎

给定正则表达式后，程序对表达式语法分析，建立语法分析数，再根据这个分析树结合正则表达式引擎生成执行程序（又称状态机、状态自动机），用于匹配字符。

正则表达式引擎有两种：

* DFA 自动机：Deterministic Final Automata，确定有限状态自动机，匹配时间复杂度 O\(n\)
* NFA 自动机：Non deterministic Final Automaton，非确定有限状态自动机，匹配时间复杂度 O\(ns\)，s 为状态数；优点是支持功能更多，如捕获 group、环视、占有有限量词等，所以**编程语言里面一般用 NFA 实现**。

### 回溯问题

假设字符串为 ”abbc“，正则表达式为 "ab{1,3}c"，则匹配过程如下图：

![](../../.gitbook/assets/image%20%28112%29.png)

NFA 自动机默认情况时贪婪模式，即匹配尽量多的内容，在第 2 步匹配到一个 b 后，会继续尽量匹配到 3 个 b。所以第 4 步，想要匹配第三个 b 时匹配不到，就会发送回溯，已经读取到 c 会被回退，指针重新指向第三个字符 b，然后再匹配 c。

#### 贪婪模式（Greedy）

如果单独使用 +、?、\*、{min,max}，则会尽量匹配多的内容。

#### 懒惰模式（Reluctant）

NFA 会尽量匹配少的内容。通过增加 ? 开启。

#### 独占模式（Possessive）

与贪婪模式一致，会尽量匹配多的内容，但是若匹配失败，不会回溯，直接匹配结束。通过增加 + 开启。

#### 案例

```java
public static void main(String[] args) {
    String text = "abbbc";
    String reg1 = "ab{1,3}bbc";
    String reg2 = "ab{1,3}?bbc";
    String reg3 = "ab{1,3}+bbc";
    System.out.println(String.format("文本: \"%s\"", text));
    System.out.println(String.format("贪婪模式: \"%s\"", reg1));
    System.out.println(String.format("懒惰模式: \"%s\"", reg2));
    System.out.println(String.format("独占模式: \"%s\"\n", reg3));
    print(text, reg1, reg2);
    Pattern p = Pattern.compile(reg3);
    Matcher matcher = p.matcher(text);
    System.out.println(String.format("\n独占模式匹配结果：%b", matcher.find()));
}

private static void print(String text, String regex1, String regex2) {
    System.out.println("regex1     regex2   percentage");
    Pattern p1 = Pattern.compile(regex1);
    Pattern p2 = Pattern.compile(regex2);
    for (int i = 0; i < 10; i++) {
        long t1 = cost(text, p1);
        long t2 = cost(text, p2);
        if (i >= 1) {
            System.out.println(String.format("%4d%10d%10.2f", t1, t2, t2 * 1.0 / t1));
        }
    }
}

private static long cost(String text, Pattern p) {
    long start = System.currentTimeMillis();
    for (int i = 0; i < 10000000; i++) {
        Matcher matcher = p.matcher(text);
        boolean b = matcher.find();
    }
    return System.currentTimeMillis() - start;
}
```

输出结果：

```text
文本: "abbbc"
贪婪模式: "ab{1,3}bbc"
懒惰模式: "ab{1,3}?bbc"
独占模式: "ab{1,3}+bbc"

regex1     regex2   percentage
 873       561      0.64
 851       587      0.69
 876       624      0.71
 850       619      0.73
 857       615      0.72
 863       623      0.72
 865       624      0.72
 858       608      0.71
 853       611      0.72

独占模式匹配结果：false
```

### 优化

#### 少用贪婪模式，多用独占模式

#### 减少分支选择

"\(x\|y\|z\)" 会降低性能，可以做如下优化：

* 常用的放在前面，就可以较快被匹配到。
* "\(abcs\|abef\)" 改成 "ab\(cd\|ef\)"。
* "\(x\|y\|z\)" 可以改成调用三次 String.indexOf\(\) 方法。

#### 减少捕获组

正则表达式中，子表达式匹配的内容保存到以数组编号或显示命名的数组中，方便后面引用，叫做捕获组，一般 \(\) 表示一个捕获组，捕获组可以嵌套。

可以用 \(?:exp\) 来表示不用捕获组。

减少捕获组可以提高性能。

## List

List 主要有 [ArrayList](../class-libraries/collection.md#arraylist-yuan-ma-fen-xi) 和 [LinkedList](../class-libraries/collection.md#linkedlist-yuan-ma-fen-xi) 两个实现类。\`

* 初始化，ArrayList 最好指定容量，避免扩容复制。
* 新增元素，不考虑扩容时：
  * 头部增加： LinkedList 效率较高，因为 ArrayList 需要移动大部分数据。
  * 中间增加：差不多，因为 ArrayList 需要移动大约一半数据，LinkedList 找到中间位置需要遍历大约一半的数据。
  * 尾部增加：ArrayList 效率较高，因为 LinkedList 的指针变换比较耗时。
* 删除元素：
  * 头部删除：LinkedList 效率较高。
  * 中间删除：差不多。
  * 尾部删除：ArrayList 效率较高。
* 遍历元素：
  * for\(;;\) 遍历：ArrayList 效率高，因为 LinkedList 每次都要找到位置。
  * 迭代器遍历：差不多。

{% hint style="warning" %}
使用迭代器遍历时，若需要删除元素，应该使用 Iterator.remove\(\)，而不是 List.remove\(\)，原因详见[迭代器（源码分析）](../class-libraries/collection.md#iterator-yuan-ma-fen-xi)。
{% endhint %}

## Stream

[Stream](../class-libraries/stream.md) 是 Java 8 新增的 Api。

* 若循环次数较少，常规的迭代性能较好。
* 单核 CPU，常规迭代性能较好。
* 循环次数较多，多核 CPU，Stream 并行性能优势明显。

所以使用 Stream 未必可以使系统性能更佳，需要结合使用场景，合理地使用 Stream。

## HashMap

[HashMap ](../class-libraries/collection.md#hashmap-yuan-ma-fen-xi)在 Java 1.8 中做了较大的优化。可根据场景合理设置初始容量和加载因子两个参数：

* 若查询较为频繁，可以减小加载因子。
* 若内存利用率需要较高，可以增大加载因子。

设计合理的 hashCode 方法，降低 hash 冲突。

若预知存储数据量，提前设置好初始容量（预知数据量 / 加载因子），可以减少 resize 操作。

## I/O

Java 有[普通 I/O](../class-libraries/java-io.md) 实现，这种实现需要把内核空间的内存与用户空间的内存相互复制，见 [Unix IO 模型](../class-libraries/java-nio.md#2-unix-io-mo-xing)，而且是阻塞的。JDK 1.4 发布了 [NIO（new I/O](../class-libraries/java-nio.md#3-java-nio)），优化了内存复制与阻塞问题。JDK 1.7 发布了 NIO2，从操作系统层面实现了异步 I/O。

* NIO 有两个重要的组件 [Channel](../class-libraries/java-nio.md#3-2-channel) 和 [Buffer](../class-libraries/java-nio.md#3-1-buffer)，可以使用它们优化读写。
* NIO 还提供了 [DirectBuffer](../class-libraries/java-nio.md#directbuffer)，可以优化内存复制。
* 使用[多路复用](../class-libraries/java-nio.md#duo-lu-fu-yong-io)模型，即 [Selector](../class-libraries/java-nio.md#selector)。

## NIO

* 选择合适的网络 IO 模型，如 [Selector](../class-libraries/java-nio.md#selector) 模式。
* 采用零拷贝技术，Linux 的 mmap可以创建一个用户空间和内和空间共享的内存块，epoll 就是使用了 mmap 减少了内存拷贝。在 Java 中则使用 [DirectBuffer](../class-libraries/java-nio.md#directbuffer)。
* 线程模型优化，如 Reactor 模型，[Redis 就采用了此模型](../../database/basic.md#xian-cheng-mo-xing)：
  * 事件接收器 Acceptor。
  * 事件分离器 Reactor
  * 事件处理器 Handler。

Reactor 模型有多种实现。

### 单线程 Reactor 模型

最开始 NIO 基于单线程实现，读写还是处于阻塞状态。

![](../../.gitbook/assets/image%20%2824%29.png)

### 多线程 Reactor 模型

Tomcat 和 Netty 都使用了一个 Acceptor 线程来监听连接请求事件，当连接成功后，会将建立的连接注册到多路复用器中。

![](../../.gitbook/assets/image%20%28189%29.png)

### 主从 Reactor 模型

这种情况下，Acceptor 不再是一个单独的线程，而是一个线程池。

![](../../.gitbook/assets/image%20%2874%29.png)

### Tomcat 调优

Tomcat NIO 有 Poller 线程池，Acceptor 接收到连接后，先将请求发送给 Poller 缓冲队列，在 Poller 中维护了一个 Selector 对象，Poller 从队列中取出连接后，注册到 Selector。

![](../../.gitbook/assets/image%20%28148%29.png)

* acceptorThreadCount：默认 1。
* maxThreads：Worker 线程池数量，默认 200。
* acceptCount：队列大小。
* maxConnections：NIO 时，应该比 maxThreads 大很多。

## 序列化

Java 提供了内置的序列化方式，即[ ObjectOutputStream 和 ObjectInputStream](../class-libraries/java-io.md#dui-xiang-xu-lie-hua)，但是性能很差，所以可以使用第三方序列化框架，如 Protobuf、Kryo、FastJson 等。

## RPC

微服务的核心是远程通信和服务治理，在满足一定服务治理的需求下，远程通信的性能对整个系统的性能至关重要。

RPC（Remote Process Call），即远程服务调用。RPC 的优化方法：

* 选择合适的通信协议：为了保证数据的可靠性，我们一般会采用 TCP；若是局域网，并对可靠性没有要求，可采用 UDP。
* 使用单一长连接
* 优化 Socket 通信：
  * 非阻塞 I/O，如 Selector。
  * Reactor 线程模型。
  * 零拷贝，如 DirectByteBuffer。
* 编码、解码，如采用 Protobuf。
* Linux 的 TCP 参数优化。
* 定义合理的报文格式

