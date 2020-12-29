# Monitor System

在性能分析时，很多时候发生性能瓶颈是突发的，到时候再登录机器场景已经消失了，所以需要通过监控、日志等方式保留现场。

## USE 法

用于性能监控，Utilization Saturation and Errors。即使用率、饱和度、错误数。

USE 只关注提现系统性能瓶颈的指标。其它指标有时也很重要，如缓存、IOPS 等。

![](../../.gitbook/assets/image%20%28325%29.png)

{% hint style="info" %}
除了 USE 原则，还有偏重于应用的 RED 原则：Rate、Error、Duration。
{% endhint %}

## 系统监控

有很多开源的监控工具，如 Zabbix、Nagios、Prometheus，以 Prometheus 为例：

![](../../.gitbook/assets/image%20%28324%29.png)

