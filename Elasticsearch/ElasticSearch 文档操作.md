本文主要是介绍 ElasticSearch 的文档增删改查和批量操作，同时会介绍一些 REST API 返回状态码的具体含义。

我们先来看下这个表：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200229213728168.png)

这个表包含了 Index、Create、Read、Update、Delete，我们先来看下 CRUD 操作的 HTTP 请求都长什么样子？

首先是提供一个 HTTP 的 method，后面是索引的名字，在 7.0 之后所以的 Type 都用 `_doc` 表示，后面是文档 id。

Create 支持两种方式，第一种是让程序自动指定文档 id，另一种是通过调用 `post /users/_doc` 去让 ES 自动生成 document id。

Get 方法只需要 Get + 索引名称 + _doc + 文档 id，通过执行这个命令就可以知道文档的具体信息了。

Index 


404 顾名思义就是文档没有找到，ES 当你在返回 REST API 的时候其实还会有各种各样的返回，下面通过一个表格了解下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020022921484959.png)

参考文献

https://www.elastic.co/guide/en/elasticsearch/reference/7.1/docs.html