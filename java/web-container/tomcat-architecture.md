# Tomcat Architecture

[Web 容器](./)一节已经讲到 Tomcat 需要实现两个核心功能：

* 处理 Socket 连接，把网络字节流与 ServletRequest 和 ServletResponse 两个对象相互转换。由**连接器（Connector）**负责。
* 加载和管理 Servlet，处理 Request 请求。由**容器（Container）**负责。

Tomcat 支持多种 I/O 模型和应用层协议。I/O 模型有：[NIO](../class-libraries/java-nio.md#3-java-nio)、NIO2、APR，应用层协议有：[HTTP 1.1](../../computer-science/network-protocol/application-layer.md#http)、AJP、HTTP 2。所以为了实现多种 I/O 模型和应用层协议，一个容器可能对接多个连接器。

连接器和容器需要组装起来才能工作，通过 Service 在连接器和容器外面包一层组装。一个 Tomcat 可以有多个 Service，可以实现通过不同的端口号来访问同一机器上部署的不同应用。

![](../../.gitbook/assets/image%20%2851%29.png)

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

![](../../.gitbook/assets/image%20%2845%29.png)

Tomcat 设计了 ProtocolHandler 来组合 Endpoint 和 Processor。

![](../../.gitbook/assets/image%20%2879%29.png)

ProtocolHandler 都有对每一种应用层协议有一层抽象，每一种 IO 模型都有具体的实现。

![](../../.gitbook/assets/image%20%2895%29.png)

#### EndPoint

是 Socket 接受和发送的处理器，负责传输层（TCP/IP）的通信。

![](../../.gitbook/assets/image%20%28100%29.png)

有两个重要的组件：

* Acceptor：用于监听 Socket 请求。
* SocketProcessor：用于处理收到的 Socket 请求，会被提交到线程池来执行。

![](../../.gitbook/assets/image%20%2896%29.png)

![](../../.gitbook/assets/image%20%2863%29.png)

#### Processor

用于处理应用层协议，即 HTTP。

![](../../.gitbook/assets/image%20%287%29.png)

EndPoint 接收到 Socket 连接后，生成一个 SocketProcessor 任务到线程池，SocketProcessor 的 run 方法会调用 Processor 组件把字节流转换成 Tomcat 的 Request。

### Adapter

ProtocolHandler 得到 Tomcat 的 Request，Processor 调用 CoyoteAdapter 的 service 方法，把 Tomcat 的 Request 转成 ServletRequest，再调用容器的 service 方法。

![](../../.gitbook/assets/image%20%2822%29.png)

## Container

![](../../.gitbook/assets/image%20%28116%29.png)

* Servlet：一个 Servlet 对象。
* Context：一个 Web 应用程序，包含多个 Servlet。
* Host：一个虚拟主机，可以部署多个应用。
* Engine：引擎，用来管理多个虚拟主机。
* Container：一个容器只能有一个引擎，所以一个 Service 也只能有一个引擎。

```markup
<!-- server.xml -->
<Server> // 顶层组件，可以包含多个Service
    <Service> // 可以包含一个 Engine，多个Connector
        <Connnector>
        </Connector>
        <Engine> // 可以包含多个 Host
            <Host> // 可以包含多个 Context
                <Context>
                </Context>            
            </Host>
        </Engine>
    </Service>
</Server>
```

可以看到容器之间是树形结构，所以 Tomcat 采用[组合模式](../../computer-science/design-patterns/composite.md)来管理这些容器。

* Container：对应 Component。
* Wrapper：对应 Leaf。
* Context、Host、Engine：对应 Composite。

```java
public interface Container extends Lifecycle {
    public Container getParent();
    public void setParent(Container container);
    public void addChild(Container child);
    public Container findChild(String name);
    public void removeChild(Container child);
}
```

### 如何定位到 Servlet

通过 Mapper 组件来实现，Mapper 保存了配置信息，如 Host 配置的域名、Context 配置的应用路径、Wrapper 配置的 Servlet 映射路径，也就是一个多层次的 map。

例子，两个域名、四个应用，如下图：

1. 根据端口和协议匹配 Service 和 Engine。
2. 根据域名匹配 Host。
3. 根据 URL 匹配 Context。
4. 根据 URL 匹配 Wrapper。

![](../../.gitbook/assets/image%20%2853%29.png)

对于一个请求，Adapter 会调用容器的 service 方法，每一层容器都会处理一些事情，所以Tomcat 使用了[责任链模式](../../computer-science/design-patterns/chain-of-responsibility.md)来实现。关键类有 Valve 和 Pipeline。

```java
public interface Valve {
  public Valve getNext();
  public void setNext(Valve valve);
  public void invoke(Request request, Response response)
}

public interface Pipeline extends Contained {
  public void addValve(Valve valve);
  public Valve getBasic();
  public void setBasic(Valve valve);
  public Valve getFirst();
}
```

每个容器都有一个 Pipeline 对象，只要调用 getFirst 方法，并调用 invoke 就可以调用这个 Pipeline 的所有 Valve。

不同层的容器通过调用 getBasic 方法，BasicValve 表示 Pipeline 的末端，负责调用下层容器 Pipeline 的第一个 Valve。

![](../../.gitbook/assets/image%20%2891%29.png)

整个过程开端于：

```java
connector.getService().getContainer().getPipeline().getFirst().invoke(request, response);

public class CoyoteAdapter implements Adapter {
    public void service(org.apache.coyote.Request req, org.apache.coyote.Response res)
            throws Exception {
        connector.getService().getContainer().getPipeline().getFirst().invoke(
                        request, response);            
    }
}
```

Wrapper 容器的 Pipeline 的最后一个 Valve 会创建一个 Filter 链，并调用 Filter 方法，最终调用到 Servlet 的 service 方法。

Valve 和 Filter 的区别：

* Valve 是 Tomcat 自定义的，与 Tomcat 紧耦合，而 Filter 是 Servlet 规范。
* Valve 工作在容器级别，拦截所有的请求；Filter 工作在应用级别，拦截某个 Web 应用的所有请求。

## LifeCycle



