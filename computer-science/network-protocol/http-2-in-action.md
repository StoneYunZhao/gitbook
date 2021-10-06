---
description: 《HTTP/2 in Action》by Barry Pollard 的读书笔记。
---

# HTTP/2 in Action

## Part 1. 向 HTTP/2 靠拢

### Chapter 1. 万维网与 HTTP

#### 1.1 万维网的原理

因特网使用 IP（Internet Protocol）协议，万维网（World Wide Web）有三项核心技术，HTTP、URL、HTML。

![](../../.gitbook/assets/image%20%28336%29.png)

#### 1.2 什么是 HTTP

HTTP 代表超文本传输协议。基于可靠的网络连接，通常为 TCP/IP。是一个请求-响应协议。

#### 1.3 HTTP 的语法和历史

* HTTP/0.9: 没有 header、body，每个请求关闭连接，错误用 HTML 响应文本。
* HTTP/1.0: 更多请求方法，添加版本号字段，首部，整数响应状态码
* HTTP/1.1: 对 1.0 的标准化和完善。强制添加 Host header，持久连接（默认），管道（不流行，且支持较差），cookie。

#### 1.4 HTTPS

HTTP 的安全版本，加密、完整性校验、身份验证。使用 TLS（Transport Layer Security）加密，TLS 的前身是 SSL（Secure Sockets layer）。

HTTPS的一个重大问题是，它只保证你正在连接到该服务器，而不能保证服务器值得信任。

#### 1.5 查看、发送和接收 HTTP 消息的工具

浏览器开发者。

Postman，curl，wget。

### Chapter 2. 通向 HTTP/2 之路

#### 2.1 HTTP 和当前的万维网

在一个连接中，请求 1-&gt; 响应 1 -&gt; 请求 2 -&gt; 响应 2 的工作方式是 HTTP 慢的根本原因，即为了等待响应会阻塞发送，在当前请求完成之前，无法发送另一个请求。

HTTP/1.1 引入管道化，连续发出多个请求，并按照顺序接收响应。但是很难实现，没有一个主流的 Web 浏览器支持管道化技术。而且存在队头（HOL）阻塞问题。

#### 2.2 解决 HTTP/1.1 性能问题的方案

大致分为两类：

* 使用多个 HTTP连接
* 合并 HTTP 请求

使用多连接与管道化相比，不会有 HOL 阻塞问题。多数浏览器可以为每个域名分配 6 个连接。域名分片技术可以开启更多连接。但是也有缺点，建立连接需要时间，维护连接需要 CPU 和内存，TCP 的慢启动算法导致每个连接初始的拥塞窗口都很小。

发送更少的请求，精灵图，CSS 和 JavaScript 文件合并，最小化文件。缺点是引入了复杂度；合并文件会导致浪费，因为不是所有的网站都需要合并文件的所有内容；缓存。

#### 2.3 HTTP/1.1 的其他问题

请求和首部是文本形式，对人类友好，但对机器不友好。

首部内容重复，尤其是 cookie。

#### 2.5 从 HTTP/1.1 到 HTTP/2

SPDY：流多路利用，请求优先级，HTTP 首部压缩，服务器推送。二进制协议。

HTTP/2 是基于 SPDY 的标准化版本。

#### 2.6 HTTP/2 对 Web 性能的影响

大多数网站可以带来很大性能提升。但不能解决所有的性能问题。

### Chapter 3. 升级到 HTTP/2

#### 3.1 HTTP/2 的支持

几乎每个现代浏览器都支持 HTTP/2，仅支持基于 HTTPS 的 HTTP/2 是浏览器厂商遵循的事实上的标准。但是中间代理若不支持，可能降级为 1.1。

几乎所有的服务器也都支持 HTTP/2 了，但是服务器升级比较麻烦，且升级 SSL/TLS 库更麻烦。

如果中间有一环不能支持 HTTP/2，可以降级到 1.1。

#### 3.2 网站开启 HTTP/2 的方法

Web 服务器开启 HTTP/2。

在 Web 服务器前放置一个 HTTP/2 的反向代理服务器，将 HTTP/2 转换为 HTTP/1.1 发送给 Web 服务器。

通过 CDN 实现 HTTP/2。

#### 3.3 常见问题

Web 服务器未启用 ALPN 支持，有些浏览器仅支持 ALPN。ALPN 是 TLS 的扩展。

## Part 2. 使用 HTTP/2

### Chapter 4. HTTP/2 协议基础

#### 4.1 为什么是 HTTP/2 而不是 HTTP/1.2

因为增加了很多概念，是根本上的变化，不向前兼容。

* 二进制的、基于数据包的协议，而 HTTP/1 是完全基于文本的。
* 多路复用替代同步请求，每个帧分配一个流标识符表名属于哪个流。客户端发起的请求使用奇数流 ID，服务器使用偶数流 ID。ID 为 0 的流为管理连接的控制流。

![](../../.gitbook/assets/image%20%28337%29.png)

* 流有优先级。
* 流可以流量控制。TCP 在连接层限流，HTTP/2 在流的层面限流。
* 首部压缩，cookie、user-agent、host、accept-encoding 等基本不变。HTTP/1 仅允许压缩 body。HTTP/2 还能跨请求压缩首部。
* 服务端推送。

#### 4.2 如何创建一个 HTTP/2 连接



#### 4.3 HTTP/2 帧



### Chapter 5. 实现 HTTP/2 推送



### Chapter 6. HTTP/2 优化



## Part 3. HTTP/2 进阶

### Chapter 7. 高级 HTTP/2 概念



### Chapter 8. HPACK 首部压缩

## Part 4. HTTP 的未来

### Chapter 9. TCP、QUIC 和 HTTP/3



### Chapter 10. HTTP 将何去何从



