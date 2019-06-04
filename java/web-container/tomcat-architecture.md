# Tomcat Architecture

[Web 容器](./)一节已经讲到 Tomcat 需要实现两个核心功能：

* 处理 Socket 连接，把网络字节流与 ServletRequest 和 ServletResponse 两个对象相互转换。由**连接器（Connector）**负责。
* 加载和管理 Servlet，处理 Request 请求。由**容器（Container）**负责。

Tomcat 支持多种 I/O 模型和应用层协议。I/O 模型有：[NIO](../class-libraries/java-nio.md#3-java-nio)、NIO2、APR，应用层协议有：[HTTP 1.1](../../computer-science/network-protocol/application-layer.md#http)、AJP、HTTP 2。所以为了实现多种 I/O 模型和应用层协议，一个容器可能对接多个连接器。

连接器和容器需要组装起来才能工作，通过 Service 在连接器和容器外面包一层组装。一个 Tomcat 可以有多个 Service，可以实现通过不同的端口号来访问同一机器上部署的不同应用。

![](../../.gitbook/assets/image%20%2845%29.png)

## Connector

连接器对容器屏蔽了 IO 模型和应用层协议，转化成标准的 ServletRequest 对象给容器。连接器的工作流程主要为以下几步：

1. 监听网络端口。
2. 接受网络连接请求。
3. 读取网络字节流。
4. 根据应用层协议（HTTP/AJP）解析字节流，生成 TomcatRequest 对象。
5. TomcatRequest 转换成 ServletRequest。
6. 调用 Servlet 容器得到 ServletResponse。
7. ServletResponse 转换成 TomcatResponse。
8. TomcatResponse 转换成字节流。
9. 把字节流返回给客户端。

Tomcat 设计了 Endpoint、Processor、Adapter 来实现上面流程。

## Container

