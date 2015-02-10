# bulk更方便

跟`mget`允许我们一次取回多个文档一样，API `bulk` 能让我们仅用一步就发起`create`，`index`，`update`或`delete`请求。如果你需要索引一个数据流，比如能批量索引和进入队列的日志事件，这就非常有用了。

`bulk`请求体的格式略微不同寻常，如下所示：

```js
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
...
```

这个格式像是 _一连串_ 用换行符`"\n"`连缀的单行json文档。请注意两点：

* 每行必须以换行符`"\n"`结尾， _包括最后一行_ 。换行符被用作高效地进行按行切分的标志。

* 一行中不能包含未转义的换行符，它们可能影响到解析，也就是说JSON不一定是打印友好的。

提示：在《bulk格式》一章我们会解释问什么API bulk会使用这种格式。

_action/metadata_ 这一行时指定对 _哪个文档_ 做 _什么动作_ 的。

_action_ 必须是`index`，`create`，`update`或`delete`之一。 _metadata_ 必须指定要被索引、创建、更新或删除的`_index`，`_type`和`_id`。

举例来说，一个`delete`请求看起来可能会是这样的。

```js
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }}
```

_请求体_ 部分由文档`_source`自己组成（文档包含的域和值）。这在`index`和`create`操作时被要求提供，因为你必须把文档提供给索引。

`update`操作也是要求请求体的，并且应该由相同的你要传给API `update`的请求体组成：`doc`，`upsert`，`script`等等。delete操作不需要`请求体`。

```js
{ "create":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
```

如果`_id`没有指定，ID就会自动生成。

```js
{ "index": { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
```

当把这些都放在一起时，一个完整的`bulk`请求就是这种格式：

```js
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} (1)
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } (2)
```

1. 注意`delete` _操作_ 没有 _请求体_ ，另一个 _操作_ 是紧跟着的。
2. 记得最后一个换行符。

Elasticsearch的响应包含了`items`数组，这个数组以我们请求的顺序列出了每个请求的结果：

```js
{
   "took": 4,
   "errors": false, (1)
   "items": [
      {  "delete": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 2,
            "status":   200,
            "found":    true
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 3,
            "status":   201
      }},
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "EiwfApScQiiy7TIKFxRCTw",
            "_version": 1,
            "status":   201
      }},
      {  "update": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 4,
            "status":   200
      }}
   ]
}}
```

1. 所有子请求都成功完成了。

每个子请求是独立执行的，所以一个子请求失败不会影响到其他的请求。如果任意一个请求失败了，那么顶级的`error`标记会被置为`true`，错误详情会在相关的请求下面显示。

```js
POST /_bulk
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "Cannot create - it already exists" }
{ "index":  { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "But we can update it" }
```

在响应中我们可以看见文档`123` `创建`失败了，因为它已经存在了，但是同样对文档`123`的子请求`index`却是成功的：

```js
{
   "took": 3,
   "errors": true, (1)
   "items": [
      {  "create": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "status":   409, (2)
            "error":    "DocumentAlreadyExistsException (3)
                        [[website][4] [blog][123]:
                        document already exists]"
      }},
      {  "index": {
            "_index":   "website",
            "_type":    "blog",
            "_id":      "123",
            "_version": 5,
            "status":   200 (4)
      }}
   ]
}
```

1. 一个或多个请求失败了。
2. 这个请求的HTTP状态码是`409 CONFLICT`。
3. 错误信息解释了为什么请求会失败。
4. 第二个请求成功了，HTTP状态码是`200 OK`。

那也就意味着`bulk`请求不是原子的——它不能用来实现事务。每个请求时独立处理的，所以一个请求的成功或失败不会影响其他的请求。

### 不要重复

可能你会批量向同一个`索引`而且使用相同的类型录入数据。给每个文档指定相同的元数据是一种浪费。相反地，就像API `mget`，`bulk`请求接受url中默认的`/_index`和`/_index/_type`：

```js
POST /website/_bulk
{ "index": { "_type": "log" }}
{ "event": "User logged in" }
```

你仍然可以在metadata一行重写`_index`和`_type`，但是它会默认使用URL中的值。

```js
POST /website/log/_bulk
{ "index": {}}
{ "event": "User logged in" }
{ "index": { "_type": "blog" }}
{ "title": "Overriding the default type" }
```

### 多大是太大?

整个bulk请求都需要加载到接收我们请求的节点的内存中，所以请求越大，其他请求可用的内存就越少。`bulk`请求的大小有一个最佳值，大于这个值，性能就不会更好甚至有可能下降。

然而，最佳大小并不是一个固定的值。它完全取决于你的硬件，你的文档的大小和复杂度，你的索引和检索的负载。好在这个 _值_ 还比较容易找到：

试着批量索引一些典型的的文档，并不断增加批量大小。当性能开始下降时，批量的大小就太大了。比较好的是从1000到1500个文档之间开始，如果你的文档很大，也可以减小一些。

通常，监控bulk请求的实际大小是很有用的。一千个1KB的文档和一千个1MB的文档是大不一样的。bulk的大小比较好的是从5到15MB之间开始。