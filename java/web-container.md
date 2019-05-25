# Web Container

Sun 公司推出了 Servlet 技术，Servlet 没有 main 方法，必须把它部署在 Servlet 容器中，Servlet 容器一般也具有 HTTP 服务器的功能，所以 Servlet 容器 + HTTP 服务器 = Web 容器。

![](../.gitbook/assets/image%20%2887%29.png)

了解 Web 容器当然要先了解 [HTTP 协议](../computer-science/network-protocol/application-layer.md#http)。

Web 容器主要做的工作是：接受连接、解析请求数据、处理请求、发送响应。

