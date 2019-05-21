# Application layer

## DHCP

**动态主机设置协议**（**D**ynamic **H**ost **C**onfiguration **P**rotocol，**DHCP**）是一个局域网的网络协议，使用UDP协议工作，主要有两个用途：

* 用于内部网或网络服务供应商自动分配 IP 地址给用户。
* 用于内部网管理员作为对所有计算机作中央管理的手段。比如 PXE。

DHCP 是 BOOTP 的增强版。

![](../../.gitbook/assets/image%20%2877%29.png)

### Discover

当一台新机器 A 加入网络的时候，仅知道自己的 MAC 地址，所以需要广播请求 IP。

<table>
  <thead>
    <tr>
      <th style="text-align:center">&#x5934;</th>
      <th style="text-align:center">&#x5185;&#x5BB9;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">MAC &#x5934;</td>
      <td style="text-align:center">
        <p>A &#x7684; MAC</p>
        <p>&#x5E7F;&#x64AD;&#x7684; MAC&#xFF08;ff:ff:ff:ff:ff:ff&#xFF09;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">IP &#x5934;</td>
      <td style="text-align:center">
        <p>A &#x7684; IP&#xFF1A;0.0.0.0</p>
        <p>&#x5E7F;&#x64AD; IP&#xFF1A;255.255.255.255</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">UDP &#x5934;</td>
      <td style="text-align:center">
        <p>&#x6E90;&#x7AEF;&#x53E3;&#xFF1A;68</p>
        <p>&#x76EE;&#x6807;&#x7AEF;&#x53E3;&#xFF1A;67</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">BOOTP &#x5934;</td>
      <td style="text-align:center">Boot request</td>
    </tr>
    <tr>
      <td style="text-align:center"></td>
      <td style="text-align:center">&#x6211;&#x7684; MAC &#x662F;&#x8FD9;&#x4E2A;&#xFF0C;&#x6211;&#x8FD8;&#x6CA1;&#x6709;
        IP</td>
    </tr>
  </tbody>
</table>### Offer

DHCP 会收到这个请求，发现这机器 A 的 MAC 没有 IP 地址，所以租给它一个 IP。

<table>
  <thead>
    <tr>
      <th style="text-align:center">&#x5934;</th>
      <th style="text-align:center">&#x5185;&#x5BB9;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">MAC &#x5934;</td>
      <td style="text-align:center">
        <p>DHCP Server &#x7684; MAC</p>
        <p>&#x5E7F;&#x64AD;&#x7684; MAC&#xFF08;ff:ff:ff:ff:ff:ff&#xFF09;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">IP &#x5934;</td>
      <td style="text-align:center">
        <p>DHCP Server &#x7684; IP&#xFF1A;192.168.1.2</p>
        <p>&#x5E7F;&#x64AD; IP&#xFF1A;255.255.255.255</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">UDP &#x5934;</td>
      <td style="text-align:center">
        <p>&#x6E90;&#x7AEF;&#x53E3;&#xFF1A;67</p>
        <p>&#x76EE;&#x6807;&#x7AEF;&#x53E3;&#xFF1A;68</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">BOOTP &#x5934;</td>
      <td style="text-align:center">Boot reply</td>
    </tr>
    <tr>
      <td style="text-align:center"></td>
      <td style="text-align:center">&#x8FD9;&#x662F;&#x4F60;&#x7684; MAC &#x5730;&#x5740;&#xFF0C;&#x6211;&#x7ED9;&#x4F60;&#x5206;&#x914D;&#x4E86;&#x8FD9;&#x4E2A;
        IP&#xFF0C;&#x4F60;&#x770B;&#x5982;&#x4F55;</td>
    </tr>
  </tbody>
</table>### Request

机器 A 可能收到多个 DHCP Server 的回复，它一般会选择最先到达的那个，并且会向网络发送一个 DHCP Request 广播数据包，包中包含客户端的 MAC 地址、接受的租约中的 IP 地址、提供此租约的 DHCP 服务器地址等，并告诉所有 DHCP Server 它将接受哪一台服务器提供的 IP 地址，告诉其他 DHCP 服务器撤销它们提供的 IP 地址。

由于还没有得到 DHCP Server 的最后确认，客户端仍然使用 0.0.0.0 为源 IP 地址、255.255.255.255 为目标地址进行广播。

<table>
  <thead>
    <tr>
      <th style="text-align:center">&#x5934;</th>
      <th style="text-align:center">&#x5185;&#x5BB9;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">MAC &#x5934;</td>
      <td style="text-align:center">
        <p>A &#x7684; MAC</p>
        <p>&#x5E7F;&#x64AD;&#x7684; MAC&#xFF08;ff:ff:ff:ff:ff:ff&#xFF09;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">IP &#x5934;</td>
      <td style="text-align:center">
        <p>A &#x7684; IP&#xFF1A;0.0.0.0</p>
        <p>&#x5E7F;&#x64AD; IP&#xFF1A;255.255.255.255</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">UDP &#x5934;</td>
      <td style="text-align:center">
        <p>&#x6E90;&#x7AEF;&#x53E3;&#xFF1A;68</p>
        <p>&#x76EE;&#x6807;&#x7AEF;&#x53E3;&#xFF1A;67</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">BOOTP &#x5934;</td>
      <td style="text-align:center">Boot request</td>
    </tr>
    <tr>
      <td style="text-align:center"></td>
      <td style="text-align:center">&#x6211;&#x7684; MAC &#x662F;&#x8FD9;&#x4E2A;&#xFF0C;&#x6211;&#x51C6;&#x5907;&#x79DF;&#x7528;&#x8FD9;&#x4E2A;
        DHCP Server &#x7ED9;&#x6211;&#x5206;&#x914D;&#x7684; IP</td>
    </tr>
  </tbody>
</table>### ACK

返回给客户机一个 DHCP ACK 消息包。

<table>
  <thead>
    <tr>
      <th style="text-align:center">&#x5934;</th>
      <th style="text-align:center">&#x5185;&#x5BB9;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">MAC &#x5934;</td>
      <td style="text-align:center">
        <p>DHCP Server &#x7684; MAC</p>
        <p>&#x5E7F;&#x64AD;&#x7684; MAC&#xFF08;ff:ff:ff:ff:ff:ff&#xFF09;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">IP &#x5934;</td>
      <td style="text-align:center">
        <p>DHCP Server &#x7684; IP&#xFF1A;192.168.1.2</p>
        <p>&#x5E7F;&#x64AD; IP&#xFF1A;255.255.255.255</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">UDP &#x5934;</td>
      <td style="text-align:center">
        <p>&#x6E90;&#x7AEF;&#x53E3;&#xFF1A;67</p>
        <p>&#x76EE;&#x6807;&#x7AEF;&#x53E3;&#xFF1A;68</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">BOOTP &#x5934;</td>
      <td style="text-align:center">Boot reply</td>
    </tr>
    <tr>
      <td style="text-align:center"></td>
      <td style="text-align:center">DHCP ACK</td>
    </tr>
  </tbody>
</table>### 客户端广播

最终租约达成的时候，还是需要广播一下，让大家都知道。

### 回收与续租

租期到了，管理员就要将IP收回。

客户机会在租期过去 50% 的时候，直接向为其提供 IP 地址的 DHCP Server 发送 DHCP request 消息包。客户机接收到该服务器回应的 DHCP ACK 消息包，会根据包中所提供的新的租期以及其他已经更新的 TCP/IP 参数，更新自己的配置。这样，IP 租用更新就完成了。

## PXE

DHCP 协议能给客户安装操作系统，这个在云计算领域大有用处。

![](../../.gitbook/assets/image%20%2880%29.png)

## HTTP

## HTTPS

## DNS

