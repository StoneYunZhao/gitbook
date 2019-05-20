# Network Layer

## ICMP

## IP

在 Windows 中通过 `ipconfig` 查看 IP，在 Linux 中通过 `ifconfig` （net-tools）或`ip iddr`（iproute2）查看。

```bash
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UNKNOWN qlen 1000
    link/ether 00:16:3e:0e:33:97 brd ff:ff:ff:ff:ff:ff
    inet 10.126.6.109/24 brd 10.126.6.255 scope global dynamic eth0
       valid_lft 292567783sec preferred_lft 292567783sec
```

ip addr 会显示所有的网卡，网卡可以有 IP 地址，也可以没有。上面输出 10.126.6.109 就是一个 IP 地址。

IP 地址被点分割为四个部分，每部分 8bit，所以 IP一共是 32 位。IPV6 有 128 位，比如 fe80::f816:3eff:fec7:7975/64 。

IP 地址分为 5 类：

| 类别 |  |  |  |  |  |  |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
|  A 类 | 0 | 网络号\(7位\) |  |  |  | 主机号\(24位\) |
| B 类 | 1 | 0 | 网络号\(14位\) |  |  | 主机号\(16位\) |
| C 类 | 1 | 1 | 0 | 网络号\(21位\) |  | 主机号\(8位\) |
| D 类 | 1 | 1 | 1 | 0 |  | 多播组号\(28位\) |
| E 类 | 1 | 1 | 1 | 1 | 0 | 留待后用\(27位\) |

![A&#x3001;B&#x3001;C &#x7C7B;&#x6240;&#x80FD;&#x5305;&#x542B;&#x7684;&#x4E3B;&#x673A;&#x6570;](../../.gitbook/assets/image%20%2890%29.png)

**无类别域间路由**（Classless Inter-Domain Routing、**CIDR**）将 32 位的IP 地址一分为二，前面是**网络号**，后面是**主机号**。X.X.X.X/A，意思是 32 位中，前 A 位是网络号，后 32 - A 位是主机号。10.126.6.255 是**广播地址**（一旦发送这个地址，10.126.6 网络里面的所有机器都可以收到），255.255.255.0 是**子网掩码**。将子网掩码与 IP 地址进行 AND 计算，就得到网络号。

例如，求 16.158.165.91/22 的第一个地址、子网掩码、广播地址。可以这么拆解：16.158.&lt;101001&gt;.&lt;01&gt;.91。第一个地址是16.158.&lt;101001&gt;.&lt;00&gt;.1，即16.158.164.1。子网掩码是 255.255.&lt;111111&gt;.&lt;00&gt;.0，即255.255.252.0。广播地址为 16.158.&lt;101001&gt;.&lt;11&gt;.255，即16.158.167.255。

在 IP 地址的后面有个 scope，对于 eth0 这张网卡来讲，是 global，说明这张网卡是可以对外的，可以接收来自各个地方的包。对于 lo 来讲，是 host，说明这张网卡仅仅可以供本机相互通信。

在IP地址的上一行是 link/ether 00:16:3e:0e:33:97 brd ff:ff:ff:ff:ff:ff，这个被称为**MAC地址**，是一个网卡的物理地址，用十六进制，6个byte表示。MAC地址的通信范围比较小，局限在一个子网里面。

第一行的 &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; 是 **net\_device flags**，**网络设备的状态标识**。UP 表示网卡处于启动的状态；BROADCAST 表示这个网卡有广播地址，可以发送广播包；MULTICAST 表示网卡可以发送多播包；LOWER\_UP 表示L1是启动的，也即网线插着呢。

mtu 1500 表示最大传输单元MTU为1500，这是以太网的默认值。

qdisc pfifo\_fast 表示排队规则为 pfifo\_fast。qdisc 全称是 **queueing discipline。**

* 最简单的 qdisc 是 pfifo，它不对进入的数据包做任何的处理，数据包采用先入先出的方式通过队列。
* pfifo\_fast 包括三个波段（band）。在每个波段里面，使用先进先出规则。
* 三个波段（band）的优先级不同，band 0 ~ 2 优先级依次降低。比如若 band 0 里面有数据包，系统就不会处理 band 1 里面的数据包。
* 数据包是按照服务类型（**Type of Service，TOS**）被分配到三个波段（band）里面的。TOS 是 IP 头里面的一个字段，代表了当前的包是高优先级的，还是低优先级的。

**Linux默认的逻辑是，如果是一个跨网段的调用，它便不会直接将包发送到网络上，而是企图将包发送到网关。**也就是说如果你配置了网关，Linux 会获取网关的 MAC 地址，然后将包发送出去。

