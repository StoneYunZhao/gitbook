# Tuning

## 性能指标

### 性能参考因素

* CPU：计算型引用、FullGC、无限循环、多线程大量上下文切换都可能造成 CPU 繁忙。
* 内存：
* 磁盘 I/O：
* 异常：构建异常栈非常消耗资源。
* 数据库：
* 锁竞争：

### 指标

* 响应时间：
  * 数据库响应时间
  * 服务端响应时间
  * 网络响应时间
  * 客户端响应时间
* 吞吐量
  * 磁盘吞吐量
    * IOPS：随机读写，如 OLTP 数据库、小文件存储等。
    * 吞吐量：顺序读写，如视频点播等。
  * 网络吞吐量：涉及 CPU、网卡、宽带等。
* 计算机资源使用率
  * CPU 占用率
  * 内存使用率
  * 磁盘 I/O 使用率
  * 网络 I/O 使用率
* 系统负载

{% hint style="info" %}
系统负载 vs CPU 利用率。系统负载表示正在运行或等待的进程或线程数，CPU 利用率表示单位时间内实时占用 CPU 的百分比。如计算密集型，CPU 利用率会很高，但是系统负载可能为 0.1；I/O 密集型，CPU 利用率可能很低，但是系统负载很高，因为很多线程可能在阻塞。
{% endhint %}

{% hint style="info" %}
TPS vs QPS。TPS（transaction per second），QPS（query per second）。一个事务可能包含多个请求，若一个用户操作只有一个请求，那么 TPS 和 QPS 就没有区别。
{% endhint %}

## 性能测试

* 微基准测试
* 宏基准测试

性能测试注意的问题：

* 热身
* 结果不稳定

## 性能优化

* 代码
* 设计
* 算法
* 时间换空间
* 空间换时间
* 参数调优

## 兜底策略

* 限流
* 熔断
* 自动扩容
* 提前扩容
