# Elasticsearch

## 1. API

Elasticsearch 的 API 在[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)上有很详细的介绍，我这里仅列举出平时用到的较多的 API，方便查询。

### 滚动升级集群

```bash
# 首先关闭 shard 的分配
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": "none"
  }
}

# 然后写入硬盘
POST _flush/synced

# 升级完成后打开
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.enable": null
  }
}
```

### Query API

```bash
# 时间范围查询
GET element-quantity-2/_search
{
  "query": {
    "range": {
      "timestamp": {
       "lte": "2018.11.13",
       "format": "yyyy.MM.dd"
      }
    }
  }
}

# 值的种类查询
GET element-quantity-2/_search
{
  "size": 0, 
  "query": {
    "match_all": {}
  },
  "aggs": {
    "count": {
      "cardinality": {
        "field": "databag_id"
      }
    }
  }
}
```

### Cat API

```bash
# 通用参数
## 显示标题
?v

## 选择显示哪些列
?h=A,B,C

## 排序
?s=A:desc,B

# 获取素有的节点
GET _cat/nodes

# 获取所有的 index
GET _cat/indices?v

# 获取符合某个 pattern 的 index 的 shard
GET _cat/shards/${pattern}

# 获取所有的 shard，按照 state 排序
GET _cat/shards?v&s=state

# 获取所有的 shard，显示指定列
GET _cat/shards?h=index,shard,prirep,state,unassigned.reason

# 获取特定类型的 thread_pool
GET /_cat/thread_pool/bulk,flush,refresh?v&h=node_name,name,active,queue,rejected,completed
​
# 获取所有的 thread_pool
GET /_cat/thread_pool?v&h=node_name,name,active,queue,rejected,completed
```

### Index API

```bash
# 关闭 Index
POST logstash-bfaccesslog-2018.12/_close

# 删除 Index，匹配格式
DELETE .monitoring-*

# 统一修改所有 index 的配置
PUT _settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}

# 修改某个 index 的配置
PUT portal/_settings
{
  "index": {
    "number_of_replicas":1
  }
}

# 获取所有 index 的配置
GET _settings

# 获取所有 template
GET _template

# 增加自己的 template
PUT _template/my_logstash
{
  "index_patterns": ["logstash-*"],
  "settings": {
    "number_of_shards": 1
  }
}

# 当 node 丢失的时候，shard 重新分配延迟时间
PUT _all/_settings
{
  "settings": {
    "index.unassigned.node_left.delayed_timeout": "5m"
  }
}
```

### Cluster API

```bash
# 获取所有关闭的 index
GET _cluster/state/blocks?pretty

# 获取集群的配置
GET /_cluster/settings

# 分析 Shard 分配问题
GET /_cluster/allocation/explain

# 获取各个节点上的 task
GET _tasks

# 获取所有数据节点和存储空间
GET _cat/allocation?v
```

### Shard Allocation

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/current/shards-allocation.html" %}

{% embed url="https://www.elastic.co/guide/en/elasticsearch/reference/current/disk-allocator.html" %}

* **cluster.routing.allocation.enable**
  * `all` - \(default\) Allows shard allocation for all kinds of shards.
  * `primaries` - Allows shard allocation only for primary shards.
  * `new_primaries` - Allows shard allocation only for primary shards for new indices.
  * `none` - No shard allocations of any kind are allowed for any indices.
* **cluster.routing.rebalance.enable**
  * `all` - \(default\) Allows shard balancing for all kinds of shards.
  * `primaries` - Allows shard balancing only for primary shards.
  * `replicas` - Allows shard balancing only for replica shards.
  * `none` - No shard balancing of any kind are allowed for any indices.
* **cluster.routing.allocation.allow\_rebalance**
  * `always` - Always allow rebalancing.
  * `indices_primaries_active` - Only when all primaries in the cluster are allocated.
  * `indices_all_active` - \(default\) Only when all shards \(primaries and replicas\) in the cluster are allocated.
* **cluster.routing.allocation.disk.threshold\_enabled**
  * `true` - default
  * `false`
* **index.unassigned.node\_left.delayed\_timeout**
  * 见 Index API。
  * 注意：此配置是 **index 级别**的，所以就算配置的时候指定为`_all`，**新建的** index 也不会有这个配置。可以用 template 的方式增加此配置。

