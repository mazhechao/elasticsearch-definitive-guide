# 集群健康

There are many statistics that can be monitored in an Elasticsearch cluster
but the single most important one is the _cluster health_, which reports a
`status` of either `green`, `yellow` or `red`:

```js
GET /_cluster/health
```

which, on an empty cluster with no indices, will return something like:

```js
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```
1. The `status` field is the one we're most interested in.

The `status` field provides an overall indication of how the cluster is
functioning. The meaning of the three colors are provided here for reference:



[horizontal]
`green`::   All primary and replica shards are active.
`yellow`::  All primary shards are active, but not all replica shards are active.
`red`::     Not all primary shards are active.

In the rest of this chapter we explain what _primary_ and _replica_ shards are
and explain the practical implications of each of the above colors.

在Elasticsearch集群中有许多数据可以监控，但是最重要的一个数据是_集群健康_。集群健康有`绿`，`黄`，`红`三个`状态`。

```js
GET /_cluster/health
```

在一个没有索引的空集群上，上述请求将返回类似下面的结果：

```js
{
   "cluster_name":          "elasticsearch",
   "status":                "green", <1>
   "timed_out":             false,
   "number_of_nodes":       1,
   "number_of_data_nodes":  1,
   "active_primary_shards": 0,
   "active_shards":         0,
   "relocating_shards":     0,
   "initializing_shards":   0,
   "unassigned_shards":     0
}
```
1. `status`是我们最感兴趣的。

`status`提供了集群运行的全局的指示。三种颜色的含义参考如下。

[horizontal]
`绿`:: 所有主索引分片和副本索引都是有效的.
`黄`:: 所有主索引分片都是有效的，但部分副本索引是无效的.
`红`:: 部分主索引分片是无效的。
