# Transport Layer

在 IP 的头里面包含了 8 位协议类型，即 IP 包里面的数据是 UDP 还是 TCP。

## UDP

**用户数据报协议**（**U**ser **D**atagram **P**rotocol，**UDP**，又称**用户数据包协议**）是一个简单的面向数据报的传输层协议。

![](../../.gitbook/assets/image%20%28119%29.png)

特点：

* 通讯简单，它相信网络是美好的，很容易送达数据，不会丢失。
* 信任，建立监听后，都可以给它传数据，也可以给任何人传数据，甚至同时。
* 不能控制拥塞，不管网络堵不堵，都照常发。

适用场景：

* 需要资源少，在网络情况比较好的内网，或者对于丢包不敏感的应用。如 [DHCP](application-layer.md#dhcp)、[PXE](application-layer.md#pxe)。
* 不需要一对一沟通，建立连接，而是可以广播的应用。DHCP 是一种广播形式。VXLAN 会用到组播（IP [5 类地址](network-layer.md#wu-lei-ip)中的 D 类），也是基于 UDP。
* 需要处理速度快，时延低，可以容忍少数丢包，但是要求即便网络拥塞，也不停止。

### 使用 UDP 的例子

* **QUIC**（**Quick UDP Internet Connections**，**快速 UDP 互联网连接**）是 Google 提出的一种基于 UDP 改进的通信协议，其目的是降低网络通信的延迟，提供更好的用户互动体验。
* **流媒体**：直播多使用 RTMP，是基于 TCP 的，但是容易卡。很多直播应用都基于 UDP 实现了自己的视频传输协议。
* **实时游戏**：游戏对实时要求较为严格，采用自定义的可靠 UDP 协议，自定义重传策略，能够把丢包产生的延迟降到最低，尽量减少网络问题对游戏性造成的影响。
* **IoT 物联网**：物联网终端可能是内存很小的嵌入式系统，维护 TCP 代价较大；另外物联网对实时性要求也很高。Google 旗下的 Nest 建立 Thread Group，推出了物联网通信协议Thread，就是基于 UDP 协议的。
* **移动通信**：在 4G 网络里，移动流量上网的数据面对的协议 GTP-U 是基于 UDP 的。

## TCP

**传输控制协议**（**T**ransmission **C**ontrol **P**rotocol，**TCP**）是一种面向连接的、可靠的、基于字节流的传输层通信协议。

相对于 UDP 需要解决 5 个问题：

* 顺序问题
* 丢包问题
* 连接维护
* 流量控制
* 拥塞控制

### TCP 头

![](../../.gitbook/assets/image%20%2898%29.png)

* 包的序号：可以解决乱序问题。
* 确认序号：可以解决包丢失问题。
* 状态位：SYN 是发起一个连接，ACK 是回复，RST 是重新连接，FIN 是结束连接。带状态位包的发送，会引起双方的状态变更。
* 窗口大小：可以做流量控制，通信双方各声明一个窗口，标识自己当前能够的处理能力。

### 三次握手

![](../../.gitbook/assets/image%20%2885%29.png)

{% hint style="warning" %}
为什么是三次握手？握手次数当然可以很多次，但是不管多少次也不能保证真的可靠。所以为了权衡，三次握手的完成之后，双方都有一次发送和返回就可以了。
{% endhint %}

{% hint style="danger" %}
注意序号，不能从 1 开始，因为可能会出现问题。这个序号是随时间变化的，每 4ms 加 1。
{% endhint %}

### 四次挥手

![](../../.gitbook/assets/image%20%28148%29.png)

{% hint style="warning" %}
B 在 ACK 之后进入 CLOSED-WAIT 状态，不能直接关闭。因为 A 已经把数据发送完了，但 B 的数据还不一定发送完成，此时 B 还是可以发送数据的。
{% endhint %}

{% hint style="warning" %}
A 发送 ACK 之后不能直接关闭，需要进入 TIME-WAIT 状态。原因是：假设没有 TIME-WAIT 状态，若 B 收不到最后这个 ACK，那么 B 会重复发送 FIN，此时 A 已经 COLED 了，那么 B 就永远收不到 ACK 了。
{% endhint %}

等待的时间设为 2 MSL（Maximum Segment Lifetime，报文最大生存时间），它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。

### 状态机

![](../../.gitbook/assets/image%20%28206%29.png)

### 滑动窗口

滑动窗口是依据接收端的处理能力的。

![](../../.gitbook/assets/image%20%2835%29.png)

**AdvertisedWindow**：接收端会给发送端的 ACK 里面带一个 ，表示接收端所能处理的事情。

**接受已确认**：表示已经接受了，也发了 ACK，但是还没被应用层读取的部分。

**累计应答（cumulative acknowledge）**：为了保证不丢包，对于发送的包都要进行应答，但是应答不是每个都需要，而是会应答某个之前的 ID，表示都收到了。

**等待接受未确认**：有两种，一种是确认没有收到包；另外一种是收到了包，但是前面的还没收到，因为累计应答机制，所以也不能应答，比如 8、9 已经收到了，但是 6、7还没收到。

**超时重试**：若上图中的 5 的 ACK 丢失了，6、7 的包丢失了，那么 5、6、7 都不能收到 ACK。每个包都设置一个超时时间，超过了这个时间，就重新尝试。若同一个包多次超时，超时时间每次都设为**两倍**。

**自适应重传算法（Adaptive Retransmission Algorithm）**：TCP 通过采样 **RTT（发送端发出一个数据报文段到收到这个数据报文段的确认的时间间隔）** 时间、RTT 和波动范围，计算出估计的超时时间，能够随着网络的变化而变化。

**快速重传**：若上图中 6、8、9 收到了，7 没收到，接收端会发送三次 6 的 ACK，发送端收到后，不会等到 7 的超时时间就会重新发送 7。

**SACK（Selective Acknowledgment）**：在 TCP 头中把缓存的地图发送给发送端，比如发送 ACK6、SACK8、SACK9，这样发送端就能看出是 7 丢失了。

**流量控制**：若接收方处理不过来，接受已确认的部分会越来越大，等待接受未确认的部分会越来越小，所以 ACK 里面的 AdvertisedWindow 会越来越小，发送端的未发送可发送的部分越来越小，流量得到了控制。

### 拥塞窗口

上节的滑动窗口 rwnd 是防止发送端把接收端塞满；而拥塞窗口（congestion window，cwnd）是防止把网络塞满，单位字节，表示在未收到接收端确认的情况下，可以连续发送的字节数。

`LastByteSent - LastByteAcked = min{cwnd, rwnd}`

由于 TCP 协议没法知道数据所经过的网络状况，所以只能通过尝试的方式逐步增加发送速度，直到出现包丢失和超时重传。

开始 cwnd 设置为 1 MSS（最大报文长度），每收到一个 ACK，`cwnd++`，即每经过一个 RTT，总的 cwnd 就加倍。

指数增长太快了，所以有一个阈值 `ssthresh = 2^16 = 65535 byte`。当超过这个值的时，每收到一个ACK：`cwnd += 1/cwnd`，所以总 cwnd 增加 1，这样就变成了线性增加。

若发生丢包情况，发送端将：`ssthresh = cwnd/2, cwnd = 1`。

可以看出若出现超时、丢包的情况，cwnd 瞬间降到 1，这种方式比较激进，将一个高速传输速度瞬间降下来，会造成网络顿卡。

**快速恢复**：上节讲到快速重传，若发现包丢失，则连续发送前一个包的三次 ACK，此时发送端会立即重新发送丢失的包，还会做另外一个件事：`cwnd = cwnd/2, ssthresh = cwnd`；然后每返回一个包：`cwnd++`。

![](../../.gitbook/assets/image%20%28149%29.png)

### 结论

* **连接问题**通过三次握手、四次挥手解决的。
* **顺序问题**、**丢包问题**、**流量控制**都是通过滑动窗口来解决的。
* **拥塞问题**是通过拥塞窗口解决的。

## 对比

* UDP 是面向无连接的；TCP 是面向连接的。
* TCP 提供可靠传输（无差错、不丢失、不重复、按序到达）；UDP 是不可靠的，可能丢失，顺序可能不一致。
* TCP 是面向字节流的，通过 TCP 自己的状态实现；UDP 是基于数据报的。
* TCP 可以控制拥塞；UDP 不能。

{% hint style="info" %}
面向连接：在互通之前，会先建立连接，比如 TCP 的三次握手。建立连接，是为了在客户端和服务端维护连接，而建立一定的**数据结构**来维护双方交互的**状态**。
{% endhint %}

## Socket 编程

UDP、TCP 协议分为客户端和服务端，写程序的时候，也有这样的区分。

Socket 可以理解为插头，双方通信需要一根线连接两个插头。Socket 作用在网络层和传输层，所以需要指定参数：

* IP 协议版本：AF\_INENT（IPv4），AF\_INENT6（IPv6）。
* 传输层协议：SOCK\_STREAM（TCP），SOCK\_DGRAM（UDP）。

### TCP 协议

![](../../.gitbook/assets/image%20%2867%29.png)

1. 应用程序通过系统调用 socket 创建一个套接字，系统给应用程序分配一个文件描述符。
2. **服务端**调用 bind\(\) ，指定参数端口和 IP，端口用于让操作系统找到你这个应用程序，IP 用于指定监听的网卡（一台机器可以有多个网卡）。
3. **服务端**调用 listen\(\)，进入 LISTEN 状态，系统创建队列。
4. **服务端**调用 accept\(\)，拿出一个已经完成的连接进行处理；若没有完成的连接，则阻塞等待。
5. **客户端**调用 connect\(\)，指定参数要连接的 IP 和端口，发起三次握手。内核给客户端分配一个临时端口。握手成功后，服务端的 accept\(\) 返回另一个 socket。
6. 建立连接后，双方调用 read\(\)、write\(\) 读写数据。

{% hint style="info" %}
* 对于第 3 点，内核中为每个 Socket 维护两个队列。一个是已经建立连接的队列，处于 ESTABLISHED 状态；一个是还没有完全建立连接的队列，处于 SYNC\_RCVD 状态。
* 对于第 4 点，监听 Socket 和真正用来传数据的 Socket 是两个，一个叫做**监听 Socket**，一个叫做**已连接 Socket**。
{% endhint %}

![](../../.gitbook/assets/image%20%28223%29.png)

Socket 在 linux 中是以文件形式的存在。每个进程都有一个数据结构 task\_struct，指向一个文件描述符数组，列出这个进程打开的所有文件的文件描述符；数组的内容是指针，指向内核中所有打开的文件列表中的某一个；文件列表中的 Socket 类型的文件也会指向一个 inode，这个 inode 不在硬盘而在内存；这个 inode 指向 Socket 在内核中的结构。

在这个结构里，主要有两个队列，一个发送队列，一个接受队列；队列保存的是缓存 sk\_buff 结构；缓存里面能看到完整的包结构。

采用 Socket 编程有如下三种情况会发生阻塞：

**connect 阻塞**：当客户端发起连接，通过调用 connect 函数，需要完成三次握手，客户端需要阻塞等待服务端发送回来的 ACK 及 SYN 信号，服务端也需要阻塞等待客户端发送的 ACK 信号。

![](../../.gitbook/assets/image%20%2829%29.png)

**accept 阻塞**：服务端调用 accept 函数，若没有新的连接进来，进程将被挂起，进入阻塞状态。

![](../../.gitbook/assets/image%20%28191%29.png)

**read、write 阻塞**：详见[ Unix IO 模型之阻塞 IO](../../java/class-libraries/java-nio.md#zu-sai-io)。

### UDP 协议

![](../../.gitbook/assets/image%20%28141%29.png)

UDP 没有连接，所以不需要三次握手，不需要 listen\(\)、accept\(\) 和 connect\(\)；仍需要 IP 和端口号，所以也需要 bind\(\)；不需要没对连接建立一组 Socket，而是只要有一个 Socket；因为没有连接，每次通信，sendto 和 recvfrom 都可以传入 IP 和端口。

## 多客户端连接

服务端需要与多个客户端连接，怎么连接多个客户端关键在于怎么处理 accept\(\) 函数返回一个已经建立连接的新 Socket。

### 多进程

accept\(\) 返回后，代表建立了一个连接，返回一个已连接的 Socket，此时通过 fork 创建一个子进程，子进程负责处理已连接的 Socket。

fork 的原理完全 copy 一个子进程，会复制文件描述符列表，也会复制内存空间，还会复制当前执行到了哪一行。复制完成后，父子进程几乎完全一样。fork 返回 0 代表子进程；fork 返回其他值，代表父进程，返回的是子进程的 PID。

![](../../.gitbook/assets/image%20%28120%29.png)

由于复制了文件描述符列表，所以父进程 accept 返回的 Socket 的文件描述符子进程也能获取到。接下来子进程就可以通过这个已连接的 Socket 与客户端交互。父进程通过子进程的 ID 查看子进程是否完成，通信完毕之后就关闭子进程。

### 多线程

多进程资源消耗太高，可以使用比进程轻量很多的线程。通过 pthread\_create 创建一个线程，在 task 列表会创建一项，文件描述符、进程空间都是共享的，只不过多了一个引用而已。

![](../../.gitbook/assets/image%20%2879%29.png)

{% hint style="info" %}
进程和线程模型都是在一个连接到来的时候创建一个进程或线程，C10K 问题表示一台机器维护一万个连接，1 亿用户需要一万台服务器，成本太高。
{% endhint %}

### IO 多路复用-select

Socket 是文件描述符，所有的 Socket 都放在一个集合 fd\_set 中。select 函数在超时时间内，监听用户感兴趣的文件描述符上可读可写或异常事件发生，一旦有发生，则轮询查询每个文件描述符，发生变化的文件描述符在 fd\_set 中设为 1，表示 Socket可读或可写，从而进行读写操作，然后再调用 select 监测下一轮变化。

```c
int select(int maxfdp1,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout)
```

可以看出 select 监视的文件有三类：writefds, readfds, expectfds。调用 select 会阻塞，当select 返回后，可通过 FD\_ISSET 遍历 fd\_set，fd\_set 可以通过以下四个宏设置：

```c
void FD_ZERO(fd_set *fdset);           // 清空集合
void FD_SET(int fd, fd_set *fdset);   // 将一个给定的文件描述符加入集合之中
void FD_CLR(int fd, fd_set *fdset);   // 将一个给定的文件描述符从集合中删除
int FD_ISSET(int fd, fd_set *fdset);   // 检查集合中指定的文件描述符是否可以读写 
```

### IO 多路复用-poll

每次调用 select，系统需要把 fd 从用户态复制到内核态。单个进程监视的 fd 数量默认是 1024。fd\_set 是通过数组实现，熟练过大导致降低效率。

poll 的机制与 select 类似，也需要把 fd 从用户态复制到内核态。但是 poll 没有最大文件描述符的限制。

### IO 多路复用-epoll

select 函数效率较低，因为每次 fd\_set 发生变化时，需要通过轮询查看集合中的每个 Socket，所以 fd\_set 的大小由 FD\_SETSIZE 控制。

epoll 函数是通过事件通知的方式，在内核中通过注册 callback 函数，当某个文件描述符发生变化的时候，就会主动通知。

通过 epoll\_create 创建一个 epoll 对象，也是一个文件，对应的文件描述符是 epoll fd，也对应着打开的文件列表中的一项，这个项目里有一颗红黑树，保存着 epoll 需要监听的所有 Socket。

epoll\_ctl 添加一个 Socket 的时候，就是往红黑树加入一项，红黑树的节点指向一个结构 epoll\_entry，这个结构挂在被监听的 Socket 的事件列表中。当 Socket 来了一个事件的时候，可以通过事件列表的 epoll 对象，调用 callback 通知 epoll。进程调用 epoll\_wait\(\) 便得到通知。

```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event event)
int epoll_wait(int epfd, struct epoll_event events,int maxevents,int timeout)
```

![](../../.gitbook/assets/image%20%28106%29.png)

这种方式能够同时监听的 Socket 特别多，epoll 解决 C10K 问题的利器。

![](../../.gitbook/assets/image%20%2819%29.png)

