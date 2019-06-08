# Tomcat Architecture

[Web 容器](./)一节已经讲到 Tomcat 需要实现两个核心功能：

* 处理 Socket 连接，把网络字节流与 ServletRequest 和 ServletResponse 两个对象相互转换。由**连接器（Connector）**负责。
* 加载和管理 Servlet，处理 Request 请求。由**容器（Container）**负责。

Tomcat 支持多种 I/O 模型和应用层协议。I/O 模型有：[NIO](../class-libraries/java-nio.md#3-java-nio)、NIO2、APR，应用层协议有：[HTTP 1.1](../../computer-science/network-protocol/application-layer.md#http)、AJP、HTTP 2。所以为了实现多种 I/O 模型和应用层协议，一个容器可能对接多个连接器。

连接器和容器需要组装起来才能工作，通过 Service 在连接器和容器外面包一层组装。一个 Tomcat 可以有多个 Service，可以实现通过不同的端口号来访问同一机器上部署的不同应用。

![](../../.gitbook/assets/image%20%2849%29.png)

## Connector

连接器对容器屏蔽了 IO 模型和应用层协议，转化成标准的 ServletRequest 对象给容器。连接器的工作流程主要为以下几步：

1. 监听网络端口。
2. 接受网络连接请求。
3. 读取网络字节流。
4. 根据应用层协议（HTTP/AJP）解析字节流，生成 Tomcat 的 Request 对象。
5. Tomcat 的 Request 转换成 ServletRequest。
6. 调用 Servlet 容器得到 ServletResponse。
7. ServletResponse 转换成 Tomcat 的 Response。
8. Tomcat 的 Response 转换成字节流。
9. 把字节流返回给客户端。

Tomcat 设计了 **Endpoint**（网络通信）、**Processor**（应用层协议解析）、**Adapter**（对象转换） 来实现上面流程。Endpoint 提供字节流给 Processor，Processor 提供 Tomcat 的 Request 给 Adapter，Adapter 提供 ServletRequest 给容器。

### ProtocolHandler

![](../../.gitbook/assets/image%20%2844%29.png)

Tomcat 设计了 ProtocolHandler 来组合 Endpoint 和 Processor。

![](../../.gitbook/assets/image%20%2874%29.png)

ProtocolHandler 都有对每一种应用层协议有一层抽象，每一种 IO 模型都有具体的实现。

![](../../.gitbook/assets/image%20%2889%29.png)

#### EndPoint

是 Socket 接受和发送的处理器，负责传输层（TCP/IP）的通信。

![](../../.gitbook/assets/image%20%2894%29.png)

有两个重要的组件：

* Acceptor：用于监听 Socket 请求。
* SocketProcessor：用于处理收到的 Socket 请求，会被提交到线程池来执行。

![](../../.gitbook/assets/image%20%2890%29.png)

![](../../.gitbook/assets/image%20%2859%29.png)

#### Processor

用于处理应用层协议，即 HTTP。

![](../../.gitbook/assets/image%20%287%29.png)

EndPoint 接收到 Socket 连接后，生成一个 SocketProcessor 任务到线程池，SocketProcessor 的 run 方法会调用 Processor 组件把字节流转换成 Tomcat 的 Request。

### Adapter

ProtocolHandler 得到 Tomcat 的 Request，Processor 调用 CoyoteAdapter 的 service 方法，把 Tomcat 的 Request 转成 ServletRequest，再调用容器的 service 方法。

![](../../.gitbook/assets/image%20%2822%29.png)

## Container

