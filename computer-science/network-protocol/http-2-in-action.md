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



### Chapter 3. 升级到 HTTP/2



## Part 2. 使用 HTTP/2

### Chapter 4. HTTP/2 协议基础



### Chapter 5. 实现 HTTP/2 推送



### Chapter 6. HTTP/2 优化



## Part 3. HTTP/2 进阶

### Chapter 7. 高级 HTTP/2 概念



### Chapter 8. HPACK 首部压缩

## Part 4. HTTP 的未来

### Chapter 9. TCP、QUIC 和 HTTP/3



### Chapter 10. HTTP 将何去何从



