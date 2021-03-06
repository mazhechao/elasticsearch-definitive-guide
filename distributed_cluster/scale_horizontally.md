# 横向扩展

当我们的应用需求增长时怎样扩容呢？如果我们启用第三个节点，我们的集群将自己重新组织，看起来就像一个`三节点集群`。

![三节点集群](../images/elas_0204.png "三节点集群")

图1. 一个三节点集群（分片被重新分配以分担负载）

各来自`节点1`和`节点2`的一个分片已经移动到了新的`节点3`，这样每个节点就只有2个分片了，而不是3个。这意味着每个节点的硬件资源（CPU，内存，I/O）被更少的分片共享了，每个分片会发挥地更好。

一个分片对它自己而言是一个完全成熟的搜索引擎，并且能够使用单个节点的所有资源。以总共6个分片（3个主和3个副本），我们的索引能够扩容至最多6个节点，每个节点一个分片，每个分片可以使用它的节点的全部资源。

