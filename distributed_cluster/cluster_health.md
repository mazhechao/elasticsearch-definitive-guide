# 集群健康

在Elasticsearch集群中有许多数据可以监控，但是最重要的一个数据是 _集群健康_ 。集群健康有`绿`，`黄`，`红`三个`状态`。

```
GET /_cluster/health
```

在一个没有索引的空集群上，上述请求将返回类似下面的结果：

```
{
   "cluster_name":          "elasticsearch",
   "status":                "green", (1)
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
1. `状态`是我们最感兴趣的。

`状态`提供了集群运行的全局的指示。三种颜色的含义参考如下。

_`绿`_ 所有主索引分片和副本索引都是有效的。  
_`黄`_ 所有主索引分片都是有效的，但部分副本索引是无效的。  
_`红`_ 部分主索引分片是无效的。

在本章剩下的部分，我们会解释什么是主分片和副本分片，并会解释上述每种颜色的实际影响。