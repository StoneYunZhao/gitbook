# Garbage Collection

## 1. 前言

### 1.1 垃圾回收内存区域

Java 自动内存管理主要针对对象的分配与回收。Java 自动回收内存最主要的区域是堆内存，Hotspot 虚拟机在1.8之前将内存回收拓展到了方法区（永久代）。

![JDK1.8 之前的堆内存示意图](<../../.gitbook/assets/image (139).png>)

{% hint style="warning" %}
上图来源网上，我认为不精确，因为永久代是方法区，方法区在 JVM 规范里面不属于堆（虽然Hotspot 可能用堆来实现的方法区）。
{% endhint %}

可以看出垃圾回收区域的分为新生代、老年代和永久代。新生代又被进一步分为：Eden 区＋Survior1 区＋Survior2 区。在 JDK 1.8中移除整个永久代，取而代之的是一个叫元空间（Metaspace）的区域（永久代使用的是JVM的堆内存空间，而元空间使用的是物理内存，直接受到本机的物理内存限制）。

### 1.2 对象分配策略

#### a. 优先在 eden 区域分配

大多数情况下，对象在新生代中 eden 区分配。当 eden 区没有足够的内存时，将先触发一次 Minor GC。

* **Minor GC**：指发生**新生代**的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快。
* **Full GC(Major GC)**：指发生在**老年代**的GC，出现了Major GC经常会伴随至少一次的Minor GC（并非绝对），Major GC的速度一般会比Minor GC的慢10倍以上。

#### b. 大对象直接进入老年代

为了避免为大对象分配内存时由于分配担保机制带来的复制而降低效率，所以直接进入老年代，比如需要大量连续内存空间的对象（字符串、数组）。

#### c. 长期存活的对象进入老年代

虚拟机给每个对象一个对象年龄（Age）计数器，如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，并将对象年龄设为1。对象在 Survivor 中每熬过一次 MinorGC，年龄就增加1，当它的年龄增加到一定程度（默认为15），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

{% hint style="info" %}
**动态年龄判断**：虚拟机不是永远要求对象年龄必须达到了某个值才能进入老年代，如果 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无需达到要求的年龄。
{% endhint %}

## 2. 对象死亡判断

### 2.1 引用计数法

给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的。

JVM 并没有使用此方法，原因是存在**循环引用**的问题。

### 2.2 可达性分析

通过一系列的称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的。

GC Roots 包括：

* 虚拟机栈中的引用的对象（本地变量和入参）。
* 方法区中类静态属性引用的对象。
* 方法区中常量引用的对象。
* 本地方法栈中 JNI 引用的对象。

![](<../../.gitbook/assets/image (6).png>)

{% hint style="warning" %}
不可达的对象，也并非是“非死不可”的，这时候它们暂时处于“缓刑阶段”，对象死亡，要经历**两次标记**过程。可达性分析法中不可达的对象被**第一次标记**并且进行一次筛选，筛选的条件是此对象是否有必要执行 finalize 方法。当对象没有覆盖 finalize 方法，或 finalize 方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行。需要执行的对象将会被放在一个队列中进行第**二次标记**，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收。
{% endhint %}

### 2.3 引用类型

![](<../../.gitbook/assets/image (250).png>)

引用计数法与可达性分析都与引用相关。JDK1.2 之后，将引用分为强引用、软引用、弱引用、虚引用。`FinalReference`为包可见，其它为 `public`。

#### a. 强引用（Strong Reference）

我们平时使用的引用即为强引用，如果对象具有强引用，那么垃圾收集器不会回收它，当内存不足时，宁愿抛出 OutOfMemoryError 异常。对于一个普通对象引用，如果没有其它引用关系，只要**超过了引用的作用域**或者**显示将引用赋值为`null`**，就可以被垃圾收集了。

#### b. 软引用（Soft Reference）

如果一个对象只具有软引用，如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。通常用来实现**内存敏感的缓存**。

{% hint style="warning" %}
软引用在最后一次引用后，还能保持一段时间，默认根据堆剩余空间计算的。可以通过参数`-XX:SoftRefLRUPolicyMSPerMB`修改。
{% endhint %}

#### c. 弱引用（Weak Reference）

在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。`WeakHashMap` 使用弱引用作为内部数据的存储方案。

#### e. 虚引用

不能通过虚引用访问对象，仅仅提供一种对象在被 finalize 后做某些事情的机制。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。**虚引用主要用来跟踪对象被垃圾回收的活动。**用`java.lang.ref.PhantomReference`表示。虚引用必须和引用队列`ReferenceQueue`关联使用。

#### f. 对象可达性状态转换

![对象可达状态流转](<../../.gitbook/assets/image (194).png>)

`java.lang.ref.Reference.get()`的实现方法里面，除了虚引用会永远返回 null，其它都可以得到原有对象，所以**软引用和弱引用都可以重新指向强引用**。

所以对于**软引用**和**弱引用**，垃圾收集器可能会存在**二次确认**，以保证没有改变为强引用。

#### g. reachabilityFence

Java 1.9 的`java.lang.ref.Reference`提供了一个新的方法`reachabilityFence`，这个底层的 API 的作用是强制使对象处于强引用状态，就算没有显示的引用指向这个对象。

```java
class Resource {
 public void action() {
     try {
         // do something
     } finally {
         // 调用 reachbilityFence，明确保障对象 strongly reachable
         Reference.reachabilityFence(this);
     }
 }
} 
```

如上面的例子，若没有调用`reachabilityFence`，则`new Resource.action()`执行后，Java 可以合法地回收这个对象，但是现在就不行了。这种书写方式在**异步编程中很常见**。

### 2.4 死亡标记与拯救

在可达性分析中不可达的对象，并不是“非死不可”，对象的死亡至少需要经历两次标记。

经过可达性分析，若对象没有与 GC Roots 相连，会被第一次标记，并判断是否需要执行 finalize() 方法（同时满足以下两个条件）：

* 是否重写了 finalize() 方法。
* 是否已经执行过 finalize() 方法。

若判断需要执行 finalize()，则对象会被放置在 F-Queue 中，稍后由一个低优先级的 Finalizer 线程在执行。

如果对象在 finalize() 方法中重新与引用链上的任何一个对象建立关联，那么对象就可以不被回收。

{% hint style="warning" %}
不建议使用 finalize() 方法，理由如下：

* 运行代价高。
* 不确定性大。
* 无法保证各对象的调用顺序。
* 基本可用 try-finally 或其它方式替代。
{% endhint %}

### 2.5 判断常量是废弃的

假如在常量池中存在字符串 "abc"，如果当前没有任何String对象引用该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池。

{% hint style="info" %}
JDK1.7及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。
{% endhint %}

### 2.6 判断类是无用的

同时满足以下三个条件：

* 该类所有的实例都已经被回收。
* 加载该类的 ClassLoader 已经被回收。
* 该类对应的 java.lang.Class 对象没有在任何地方被引用。

虚拟机**可以**对满足上述3个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样不使用了就会必然被回收。

## 3. 垃圾回收算法

### 3.1 标记-清除（Mark-Sweep）

算法分为“标记”和“清除”阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。

* 效率很高。
* 清除后有很多不连续的碎片。

![](<../../.gitbook/assets/image (109).png>)

收集器：CMS。

### 3.2 复制（Copying）

将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。

![](<../../.gitbook/assets/image (268).png>)

收集器：Serial、ParNew、Parallel Scavenge、

### 3.3 标记-整理（Mark-Compact）

一般用于**老年代**，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一段移动，然后直接清理掉端边界以外的内存。

![](<../../.gitbook/assets/image (235).png>)

收集器：Serial Old、Parallel Old

### 3.4 分代收集

根据对象存活周期的不同将内存分为几块，根据各个年代的特点选择合适的垃圾收集算法。

比如在新生代中，每次收集都会有大量对象死去，所以可以选择复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清楚”或“标记-整理”算法进行垃圾收集。

## 4. 垃圾收集器

![](<../../.gitbook/assets/image (175).png>)

### 4.1 Serial 收集器

只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ **"Stop The World"** ），直到它收集结束。常用于 **Client 模式**下的 JVM。

![](<../../.gitbook/assets/image (31).png>)

**新生代采用复制算法，老年代采用标记-整理算法。**

### 4.2 ParNew 收集器

Serial 收集器的**多线程版本**，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和Serial收集器完全一样。常用于 **Server 模式**下的 JVM，除了 Serial，只有它能与 CMS（真正意义上的**并发**收集器） 配合。

* **并行（Parallel）** ：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。
* **并发（Concurrent）**：指用户线程与垃圾收集线程同时执行（但不一定是并行，可能会交替执行），用户程序在继续运行，而垃圾收集器运行在另一个CPU上。

![](<../../.gitbook/assets/image (197).png>)

**新生代采用复制算法，老年代采用标记-整理算法。**

### 4.3 Parallel Scavenge 收集器

Parallel Scavenge 收集器关注点是**吞吐量**（高效率的利用CPU）。CMS 等垃圾收集器的关注点更多的是用户线程的**停顿时间**（提高用户体验）。

![](<../../.gitbook/assets/image (130).png>)

**新生代采用复制算法，老年代采用标记-整理算法。**

### **4.4 Serial Old 收集器**

Serial **** 收集器的老年代版本。

### 4.5 Parallel Old 收集器

Parallel Scavenge 收集器的老年代版本。使用多线程和“标记-整理”算法。在**注重吞吐量**以及 CPU 资源的场合，可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

### 4.6 CMS 收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取**最短回收停顿时间**为目标的收集器。它而非常符合在注重用户体验的应用上使用。

CMS 收集器是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（**基本上**）同时工作。

* **初始标记：** 暂停所有的其他线程，并记录下直接与 root 相连的对象，速度很快 。
* **并发标记：** 同时开启GC和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以GC线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
* **重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
* **并发清除：** 开启用户线程，同时GC线程开始对为标记的区域做清扫。

![](<../../.gitbook/assets/image (233).png>)

主要优点：**并发收集、低停顿**，缺点：

* 对CPU资源敏感；
* 无法处理浮动垃圾；
* 它使用的回收算法-“**标记-清除**”算法会导致收集结束时会有大量空间碎片产生。

### 4.7 G1 收集器

**G1 (Garbage-First)**是一款面向**服务器**的垃圾收集器，主要针对配备**多颗处理器**及**大容量内存**的机器，以极高概率满足 GC 停顿时间要求的同时，还具备高吞吐量性能特征。

它具备一下特点：

* **并行与并发**：G1能充分利用CPU、多核环境下的硬件优势，使用多个CPU（CPU或者CPU核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的GC动作，G1收集器仍然可以通过并发的方式让 Java 程序继续执行。
* **分代收集**：虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念。
* **空间整合**：与CMS的“标记--清理”算法不同，G1从**整体**来看是基于“**标记整理**”算法实现的收集器；从**局部**上来看是基于“**复制**”算法实现的。
* **可预测的停顿**：这是G1相对于CMS的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内。

G1收集器的运作大致分为以下几个步骤：

* **初始标记**
* **并发标记**
* **最终标记**
* **筛选回收**

**G1收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的Region(这也就是它的名字Garbage-First的由来)**。这种使用Region划分内存空间以及有优先级的区域回收方式，保证了GF收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。

## Reference

* [https://juejin.im/post/5b85ea54e51d4538dd08f601](https://juejin.im/post/5b85ea54e51d4538dd08f601#heading-14)
* [https://zhuanlan.zhihu.com/p/28258571](https://zhuanlan.zhihu.com/p/28258571)
* [http://www.importnew.com/26383.html](http://www.importnew.com/26383.html)





