# 容错

我们已经说过Elasticsearch可以容忍节点失败，让我们继续看看并且尝试下。如果我们杀掉第一个节点，我们的节点看起来就像一个`杀过节点的集群`。

![杀掉一个节点的集群](../images/elas_0206.png "杀掉一个节点的集群")

图1. 杀掉一个节点的集群

被我们杀掉的节点是master节点。为了正确地运行，一个集群必须有一个master节点，所以接下来的第一件事就是选举一个新的master节点：`节点2`。

当我们杀掉`节点1`时，主分片`1`和`2`就丢失了，如果缺少主分片，我们的索引就不能正常工作了。这时候如果我们检查集群健康的话，我们应该看到状态是`红`：不是所有的主分片是存活的。

幸好这两个丢失的主分片在其他节点上都有完整的拷贝，所以新的master节点做的第一件事就是促使`节点2`和`节点3`上的这些副本分片变为主分片，并将集群健康度变回`黄`。这个过程是一瞬间的，就像按一下开关一样。

那为什么我们的集群健康度是`黄`而不是`绿`呢？我们有了全部的3个分片，但是我们希望每个主分片有两个副本，而当前只分配了一个副本。这阻止了集群健康度达到`绿`，但这里我们也无需过度担心：即使我们杀掉`节点2`，我们的应用 _仍然_ 能够运行而没有数据丢失，因为`节点3`有每个分片的拷贝。

至此，你应该能清楚地知道分配是如何让Elasticsearch横向扩展以及如何保障你的数据是安全的。后面我们会更详细地研究一个分片的生命周期。
