# Open source contribution

## InfluxDB

汇总 =&gt; [link](https://github.com/influxdata/influxdb/pulls?q=is%3Apr+author%3AStoneYunZhao+)



### 2021.06.25 httpd/handler - review required

Master-1.x: [fix\(httpd\): abort processing write request when encountering a precision error.](https://github.com/influxdata/influxdb/pull/21746)

简单的小 bug。可见这么多 star 的开源项目中也有这么显而易见的疏忽。

### 2021.04.07 LocalShardMapping - merged

Master: [https://github.com/influxdata/influxdb/pull/21390](https://github.com/influxdata/influxdb/pull/21390)

没有必要再复制一份完全一样的数据。

### 2021.03.22 tsi - merged

* Master: [https://github.com/influxdata/influxdb/pull/21013](https://github.com/influxdata/influxdb/pull/21013)
* 2.0: [https://github.com/influxdata/influxdb/pull/21045](https://github.com/influxdata/influxdb/pull/21045)
* master-1.x: [https://github.com/influxdata/influxdb/issues/21044](https://github.com/influxdata/influxdb/issues/21044)

用 bitmap 的 intersect 操作替代逐个比较。

### 2021.03.09 tsmFullCompactionQueue - merged

Master: [https://github.com/influxdata/influxdb/pull/20897](https://github.com/influxdata/influxdb/pull/20897)

tsmFullCompactionQueue 统计值错误。

### 2021.03.08 Cache - Suggestion Accepted

在 [\#20843](https://github.com/influxdata/influxdb/pull/20843) 中建议删除一段代码，他们开了个 PR：

[https://github.com/influxdata/influxdb/pull/20890](https://github.com/influxdata/influxdb/pull/20890)

### 2021.03.01 WAL 限流器 - merged

[https://github.com/influxdata/influxdb/pull/20814](https://github.com/influxdata/influxdb/pull/20814)

WAL 中的 limiter 没有工作。

后续优化： [https://github.com/influxdata/influxdb/issues/21600](https://github.com/influxdata/influxdb/issues/21600)

### 2021.02.28 WAL totalOldDiskSize - merged

* master: [https://github.com/influxdata/influxdb/pull/20811](https://github.com/influxdata/influxdb/pull/20811)
* 2.0: [https://github.com/influxdata/influxdb/pull/20851](https://github.com/influxdata/influxdb/pull/20851)

WAL 中的 totalOldDiskSize 计算不对。

### 2020.12.30 Sync log - closed

[https://github.com/influxdata/influxdb/pull/20422](https://github.com/influxdata/influxdb/pull/20422)

程序退出时应该先把日志刷盘。由于代码很少，而且也不是很重要，加上他们那边一直没有 review 这个 PR，所以我关掉算了。

### 2020.11.25 Cache ring - merged

[https://github.com/influxdata/influxdb/pull/20172](https://github.com/influxdata/influxdb/pull/20172)

Cache ring 中存在数据竞争，同时修复了一些 Test。原始 PR 合并到 1.8，但是 1.8 正在排查性能问题，已经冻结，他们通过 forward-port 或 backward-port 合并到了 master-1.x、 master、2.0：

* master-1.x: [https://github.com/influxdata/influxdb/pull/20802](https://github.com/influxdata/influxdb/pull/20802)
* master: [https://github.com/influxdata/influxdb/pull/20797](https://github.com/influxdata/influxdb/pull/20797)
* master 后来又改了: [https://github.com/influxdata/influxdb/pull/20803](https://github.com/influxdata/influxdb/pull/20803)
* 2.0: [https://github.com/influxdata/influxdb/pull/20843](https://github.com/influxdata/influxdb/pull/20843)

### 2020.11.20 分片路由优化 - merged

[https://github.com/influxdata/influxdb/pull/20118](https://github.com/influxdata/influxdb/pull/20118)

当只有一个分片时，不需要计算每个 point 的 hash 值。

## NSQ

### 2021.01.11 NSQ - approved

[https://github.com/nsqio/go-diskqueue/pull/24](https://github.com/nsqio/go-diskqueue/pull/24)

为磁盘队列增加 Peek 方法。

## InfluxDB Document

汇总 =&gt; [link](https://github.com/influxdata/docs-v2/pulls?q=+is%3Apr+author%3AStoneYunZhao+)

### 2021.09.17 InfluxDB docs - merged

Master: [https://github.com/influxdata/docs-v2/pull/3143](https://github.com/influxdata/docs-v2/pull/3143)

文档超链接跳转错误。

### 2021.06.29 InfluxDB docs - merged

Master: [values of query parameter \`precision\` for write api.](https://github.com/influxdata/docs-v2/pull/2792)

小小的文档描述错误。

### 2020.11.30 InfluxDB docs - merged

[https://github.com/influxdata/docs-v2/pull/1900](https://github.com/influxdata/docs-v2/pull/1900)

对 ALTER RETENTION POLICY 的描述不严谨。

