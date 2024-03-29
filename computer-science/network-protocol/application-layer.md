# Application layer

## DHCP

**动态主机设置协议**（**D**ynamic **H**ost **C**onfiguration **P**rotocol，**DHCP**）是一个局域网的网络协议，使用UDP协议工作，主要有两个用途：

* 用于内部网或网络服务供应商自动分配 IP 地址给用户。
* 用于内部网管理员作为对所有计算机作中央管理的手段。比如 PXE。

DHCP 是 BOOTP 的增强版。

![](<../../.gitbook/assets/image (182).png>)

### Discover

当一台新机器 A 加入网络的时候，仅知道自己的 MAC 地址，所以需要广播请求 IP。

|    头    |                         内容                        |
| :-----: | :-----------------------------------------------: |
|  MAC 头  |  <p>A 的 MAC</p><p>广播的 MAC（ff:ff:ff:ff:ff:ff）</p>  |
|   IP 头  | <p>A 的 IP：0.0.0.0</p><p>广播 IP：255.255.255.255</p> |
|  UDP 头  |            <p>源端口：68</p><p>目标端口：67</p>            |
| BOOTP 头 |                    Boot request                   |
|         |                 我的 MAC 是这个，我还没有 IP                |

### Offer

DHCP 会收到这个请求，发现这机器 A 的 MAC 没有 IP 地址，所以租给它一个 IP。

|    头    |                                内容                               |
| :-----: | :-------------------------------------------------------------: |
|  MAC 头  |    <p>DHCP Server 的 MAC</p><p>广播的 MAC（ff:ff:ff:ff:ff:ff）</p>    |
|   IP 头  | <p>DHCP Server 的 IP：192.168.1.2</p><p>广播 IP：255.255.255.255</p> |
|  UDP 头  |                   <p>源端口：67</p><p>目标端口：68</p>                   |
| BOOTP 头 |                            Boot reply                           |
|         |                   这是你的 MAC 地址，我给你分配了这个 IP，你看如何                  |

### Request

机器 A 可能收到多个 DHCP Server 的回复，它一般会选择最先到达的那个，并且会向网络发送一个 DHCP Request 广播数据包，包中包含客户端的 MAC 地址、接受的租约中的 IP 地址、提供此租约的 DHCP 服务器地址等，并告诉所有 DHCP Server 它将接受哪一台服务器提供的 IP 地址，告诉其他 DHCP 服务器撤销它们提供的 IP 地址。

由于还没有得到 DHCP Server 的最后确认，客户端仍然使用 0.0.0.0 为源 IP 地址、255.255.255.255 为目标地址进行广播。

|    头    |                         内容                        |
| :-----: | :-----------------------------------------------: |
|  MAC 头  |  <p>A 的 MAC</p><p>广播的 MAC（ff:ff:ff:ff:ff:ff）</p>  |
|   IP 头  | <p>A 的 IP：0.0.0.0</p><p>广播 IP：255.255.255.255</p> |
|  UDP 头  |            <p>源端口：68</p><p>目标端口：67</p>            |
| BOOTP 头 |                    Boot request                   |
|         |      我的 MAC 是这个，我准备租用这个 DHCP Server 给我分配的 IP      |

### ACK

返回给客户机一个 DHCP ACK 消息包。

|    头    |                                内容                               |
| :-----: | :-------------------------------------------------------------: |
|  MAC 头  |    <p>DHCP Server 的 MAC</p><p>广播的 MAC（ff:ff:ff:ff:ff:ff）</p>    |
|   IP 头  | <p>DHCP Server 的 IP：192.168.1.2</p><p>广播 IP：255.255.255.255</p> |
|  UDP 头  |                   <p>源端口：67</p><p>目标端口：68</p>                   |
| BOOTP 头 |                            Boot reply                           |
|         |                             DHCP ACK                            |

### 客户端广播

最终租约达成的时候，还是需要广播一下，让大家都知道。

### 回收与续租

租期到了，管理员就要将IP收回。

客户机会在租期过去 50% 的时候，直接向为其提供 IP 地址的 DHCP Server 发送 DHCP request 消息包。客户机接收到该服务器回应的 DHCP ACK 消息包，会根据包中所提供的新的租期以及其他已经更新的 TCP/IP 参数，更新自己的配置。这样，IP 租用更新就完成了。

## PXE

DHCP 协议能给客户安装操作系统，这个在云计算领域大有用处。

![](<../../.gitbook/assets/image (188).png>)

## HTTP

HTTP 协议是浏览器与服务器之间的数据传送协议。本质是浏览器与服务器之间约定好的通讯格式。

HTTP 协议是基于 TCP 的，1.1 版本的 HTTP 模式开启了 Keep-Alive（Connection: keep-alive），可以在多次 HTTP 请求中复用一个 TCP 连接，不用每次都三次握手。

![HTTP 请求报文格式](<../../.gitbook/assets/image (171).png>)

* **方法**：GET、POST、PUT、DELETE、HEAD、PATCH 等
* **版本**：1.0、1.1、2.0
* 头部：
  * Accept-Charset：客户端可以接受的字符集
  * Content-Type：正文的格式
  * **Cache-Control**：max-age 为 0，则不用缓存； 否则若资源缓存时间小于 max-age，则可以使用缓存。
  * If-Modified-Since：若资源在某个时间之后没有更新，那么客户端就不用下载最新的资源了。

![HTTP 响应报文格式](<../../.gitbook/assets/image (198).png>)

HTTP 协议是无状态的，这样就会造成 Web 应用不知道这个请求来自哪里。

### Cookie

Cookie 是 HTTP 报文的一个请求头，Web 应用可以将一些用户信息放在 cookie 中，用户的每次请求都带上 Cookie 头。**Cookie 的本质是一份存储在用户本地的文件，每次请求都带上这份信息。**

### Session

Cookie 若明文带了用户信息，会有安全问题。所以服务端保存用户的状态，把 Session ID 返回给用户端，用户端每次请求都通过 Cookie 带上这个 Session ID。

## HTTPS

## DNS

IP 不好记，所以有了域名。域名用“.”分隔的多个单词，最右边为“顶级域名”，然后是“二级域名”，往左依次降低。最左边通常用来表示主机用途，如 www 表示万维网，mail 表示邮件。所有域名都以 (.) 作为后缀，平时一般省略。DNS 服务器一般监听端口号 53。

DNS 负责把域名解析成 IP 地址，它的核心系统是一个三层树状、分布式服务：

1. 根域名服务器（Root DNS Server）：负责顶级域名，如返回 com、cn、net 等顶级域名服务器的地址。目前全世界有 13 组根域名服务器，有数百台镜像。
2. 顶级域名服务器（Top-level DNS Server）：负责各自域名下的权威域名服务器，如 com 顶级域名服务器返回 apple.com 的地址。
3. 权威域名服务器（Authoritative DNS Server）：负责自己域名，如 apple.com权威域名服务器返回 www.apple.com 的地址。

核心 DNS 服务器不能服务全球网民，所以很多公司、网络运营商都有自己的 DNS 服务器，本质是缓存数据。操作系统也会对 DNS 的解析做缓存，另外/etc/hosts 可以自定义 DNS 记录。

DNS 服务支持多种不同类型的记录：

* A 记录：用于把域名转换成 IP 地址。
* CNAME 记录：用于创建别名。
* NS 记录：表示域名对应的域名服务器的地址。

可通过 /etc/resolve.conf 配置域名服务器，[nslookup、dig](../linux/diagnostic-tools.md#bind-utils) 查看域名。&#x20;

域名的作用：

1. 重定向，通过访问域名，可以做到对外服务不变，后端的 IP 地址可以任意改变。
2. 负载均衡。

优化：

* 使用 DNS 缓存，比如 dnsmasq。
* 对 DNS 解析的结果进行预取。这是浏览器等 Web 应用中最常用的方法，也就是说，不等用户点击页面上的超链接，浏览器就会在后台自动解析域名，并把结果缓存起来。
* 使用 HTTPDNS 取代常规的 DNS 解析。域名劫持普遍存在，使用 HTTP 协议绕过链路中的 DNS 服务器，就可以避免域名劫持的问题。
