# Tomcat Architecture

[Web 容器](./)一节已经讲到 Tomcat 需要实现两个核心功能：

* 处理 Socket 连接，把网络字节流与 ServletRequest 和 ServletResponse 两个对象相互转换。
* 加载和管理 Servlet，处理 Request 请求。

Tomcat 设计了两个核心组件连接器（Connector）和容器（Container）来分别负责这两部分功能。

Tomcat 支持多种 I/O 模型和应用层协议。I/O 模型有：[NIO](../class-libraries/java-nio.md#3-java-nio)、NIO2、APR，应用层协议有：[HTTP 1.1](../../computer-science/network-protocol/application-layer.md#http)、AJP、HTTP 2。

所以为了实现多种 I/O 模型和应用层协议，一个容器可能对接多个连接器。



## Connector

## Container

