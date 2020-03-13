# RMI

RMI（Remote Method Invocation）是 JDK 自带的 RPC 框架，是纯 Java 网络分布式应用系统的核心解决方案。

实现了一台虚拟应用对远程方法的调用可以同对本地方法的调用一样。

![&#x5B9E;&#x73B0;&#x539F;&#x7406;](../../.gitbook/assets/image%20%28157%29.png)

性能问题：

* Java 默认的序列化，性能不是很好。
* 计划于 TCP 短连接。
* 阻塞式 I/O。

