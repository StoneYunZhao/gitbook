# Preface

Java 是一门面向对象的语言，有两个显著的特征：

* 书写一次，到处运行（Write once，run anywhere）
* 垃圾收集（GC，Garbage Collection）

![Java &#x5E73;&#x53F0;&#x6982;&#x89C8;](../../.gitbook/assets/image%20%2833%29.png)

Java 分为**编译期**和**运行期**，这里的编译与 C/C++ 编译不同，Java 编译成字节码，而 C/C++ 编译成机器码。

运行期 Java 加载字节码，**解释**或**编译（注意：这里的编译又与上面说的编译不一样）**执行，如 JDK 8 是解释和编译混合执行。Server 模式的 JVM 会进行上万次调用收集足够信息来高效编译，而 client 模式是1500 次。HotSpot 内置了两个 JIT（Just-In-Time），C1 对应 client 模式，适合启动速度高的应用，如桌面应用；C2 对应 server 模式，适合长时间运行的应用，如服务端应用，采用**分层编译**（TieredComplilation）。

* `-Xint`：仅解释执行。
* `-Xcomp`：不要进行解释执行，会导致启动慢很多，而且不能根据运行情况有效优化，如分支预测。

**AOT**（Ahead-of-Time Compilation）：直接将字节码编译成机器码，如 JDK 9 引入了 AOT，增加了 jaotc 工具。**AOT 可以和分层编译结合**，不是二选一。

