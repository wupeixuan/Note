本文主要介绍什么是 ElasticSearch 以及为什么需要它，如何在本机安装部署 ElasticSearch 实例，同时会演示安装 ElasticSearch 插件，以及如何在本地部署多实例集群，方便在日后学习分布式相关原理。

# 什么是 ElasticSearch？

ElasticSearch 是一个基于 **Lucene** 的搜索服务器，它提供了一个**分布式**多用户能力的全文搜索引擎，基于 RESTful web 接口。**ElasticSearch 是用 Java 开发的**，并作为 Apache 许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便，其中维基百科、Stack Overflow、Github 的搜索都是基于 ElasticSearch 构建的。

简而言之，ElasticSearch 是一个**开源**的**近实时**的**分布式存储、搜索、分析引擎**。

ElasticSearch 的主要功能简单来说就是两方面-**搜索**和**聚合**（比如最近7天口罩商品销量排名前10的商家列表），另外当海量数据不断增长的时候，还提供分布式存储以及集群管理能力。

因为 ElasticSearch 是起源于 Lucene 的，在这里简单地介绍下 Lucene：

Lucene 就是一个 jar 包，里面包含了封装好的各种建立**倒排索引**，以及进行搜索的代码，包括各种算法。我们就用 Java 开发的时候，引入 Lucene jar，然后基于 Lucene 的 API 进行去进行开发就可以了。使用 Lucene 就可以去将已有的数据建立索引，Lucene 会在**本地磁盘**上面，给我们组织索引的数据结构。另外的话，我们也可以用 Lucene 提供的一些功能和 API 来针对磁盘上的索引数据进行搜索。

同时 Lucene 也存在着很多局限性，比如只能基于 Java 语言开发，类库的接口学习曲线陡峭，原生并不支持水平扩展等。ElasticSearch 就解决了以上存在的问题，做到了支持分布式，可水平扩展，并且降低全文检索的学习曲线，可以被任何编程语言调用。

## 为什么需要 ElasticSearch？

用数据库，也可以实现搜索的功能，为什么还需要搜索引擎呢？ 那我们来看一下如果用数据库做搜索会怎么样：

假如你在电商平台搜索物品，每个物品在数据库都有一条记录，每条记录的指定字段的文本，可能会很长，比如说商品描述字段的长度，有长达数千个，甚至数万个字符，这个时候，每次都要对每条记录的所有文本进行扫描，去判断包不包含我指定的这个关键词，比如我们搜索“口罩”，效率就会很慢。

并且还不能将搜索词拆分开来，尽可能去搜索更多的符合你的期望的结果，比如输入“医用罩”，就搜索不出来“医用口罩”。

但是基于 ElasticSearch 的 Github，比如我们搜索“设模式”，搜索结果也会出现“设计模式”：

![](https://img-blog.csdnimg.cn/20200226140055639.png)

因此，用数据库来实现搜索，是不太靠谱的，性能上也会比较差。

前面说了 ElasticSearch 是分布式搜索引擎，那么就让我们来看下 ElasticSearch 的分布式架构：

## ElasticSearch 分布式架构

![](https://img-blog.csdnimg.cn/20200225235529828.png)

ElasticSearch 就是为高可用和可扩展而生的，从图中可以看出 ElasticSearch 很容易去做水平扩展，同时也是非常容易在个人电脑上做开发环境的搭建。当数据规模变大的情况下，集群规模可以从单个扩展至数百个节点，除此之外，**ElasticSearch 还支持设置不同的节点类型**，针对日志类的应用，可以用集群做一个 Hot & Warm 部署。

> 可以通过购置性能更强的服务器来完成，称为**垂直扩展**或者**向上扩展**，或增加更多的服务器来完成，称为**水平扩展**或者**向外扩展**。

ElasticSearch 是基于 Java 语言开发的，在之前安装是需要在本机安装 JDK 开发环境，但是**在 ElasticSearch 7.0 版本后，内置了 Java 开发环境**，使得安装会变得更加简单。

接下来让我们来动手安装 ElasticSearch。

# ElasticSearch 安装与配置

官网下载地址： https://www.elastic.co/downloads/ElasticSearch

![](https://img-blog.csdnimg.cn/20200225231727897.png)

打开官网后根据自己的系统选择对应文件，因为我用的是 Windows 环境，所以下载 ElasticSearch-7.1.0-windows-x86_64.zip 版本，下载完成后解压即可。

在运行 ElasticSearch 之前，先让我们来窥探下 ElasticSearch 的文件目录结构：

## 文件目录结构

![文件目录结构](https://img-blog.csdnimg.cn/20200225231357504.png)

解压后的目录结构如上图所示，其中 bin 目录下主要是脚本文件；config 目录下主要是 ElasticSearch 配置文件，其中 ElasticSearch.yml 是主要需要配置的地方；JDK 目录是在 ElasticSearch 7.0 版本后出现的，为 Java 运行环境；data 目录其实包含了 ElasticSearch 的相关数据文件；lib 目录包含 Java 的类库；logs 目录下主要是 ElasticSearch 运行过程中所有的日志文件；modules 目录下包含所有的 ES 模块；**ElasticSearch 是可以通过插件的方式去进行扩展**，因此 plugins 目录下包含所有已安装的插件。

在 config 目录下有一个 jvm.options 文件，这是 JVM 的配置文件，7.1 版本中默认的 Xms 和 Xmx 都为 1GB。

> **建议把 Xms 和 Xmx 设置成一样的，也就是最大最小内存，Xmx 不要超过机器内存的 50%，内存的总量不要超过 30GB。**

接下来让我们启动 ElasticSearch。

## 运行单个 ElasticSearch 实例

进入 bin 目录，打开 cmd 命令行，输入 `elasticsearch -E node.name=node0 -E cluster.name=wupx -E path.data=node0_data`，就可以运行一个 ElasticSearch 实例，ElasticSearch 本身特点之一就是开箱即用，如果是中小型应用，数据量少，操作不是很复杂，直接启动就可以用了。

可以在浏览器输入 `http://localhost:9200`，就可以看到 ElasticSearch 在本机启动起来了，网页显示内容如下：

```
{
  "name" : "node0",
  "cluster_name" : "wupx",
  "cluster_uuid" : "1TT8NYjcSxmLKeG-1ukqfA",
  "version" : {
    "number" : "7.1.0",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "606a173",
    "build_date" : "2019-05-16T00:43:15.323135Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

其中 name 为 **节点名称**，cluster_name 为**集群名称**（默认的集群名称为 ElasticSearch），version.number: 7.1.0 为 **ElasticSearch 版本号**。

接下来让我们看下如何在本机安装 ElasticSearch 插件。

## 安装与查看插件

在 cmd 中输入 `elasticsearch-plugin list` 可以查看本机已安装的插件。

输入 `elasticsearch-plugin install analysis-icu` 下载国际化分词插件安装到本机。

安装成功后，启动 ElasticSearch，访问 `http://localhost:9200/_cat/plugins` ，我们可以看到这个插件成功安装在这个集群上面了。

![](https://img-blog.csdnimg.cn/20200225233951771.png)

如何在开发机上运行多个 ElasticSearch 实例呢？我们知道 ElasticSearch 其中一个特色是可以以分布式的方式去运行，也就是可以在多个机器上去运行多个不同实例来组成一个集群，为了能够理解内部工作机制，让我们一起来实践操作下。

## 运行多个 ElasticSearch 实例

在 cmd 中输入如下代码，每次启动指定节点名称，指定相同的集群名字，指定不同的存放数据地址，就可以运行四个 ElasticSearch 实例在后台。

```
elasticsearch -E node.name=node0 -E cluster.name=wupx -E path.data=node0_data -d
elasticsearch -E node.name=node1 -E cluster.name=wupx -E path.data=node1_data -d 
elasticsearch -E node.name=node2 -E cluster.name=wupx -E path.data=node2_data -d 
elasticsearch -E node.name=node3 -E cluster.name=wupx -E path.data=node3_data -d
```

在浏览器访问 `http://localhost:9200/_cat/nodes` ，可以查看集群存在哪里节点。

![](https://img-blog.csdnimg.cn/20200225234723278.png)

# 总结

这就是本文的主要内容，我相信大家对 ElasticSearch 有了初步的了解，都可以在本地运行一个 ElasticSearch 实例，也学会了在实例上安装你需要的插件，最后也实践了怎么在本机运行多个 ElasticSearch 实例的集群，这可以帮助我们以后更好地理解 ElasticSearch 分布式集群工作的方式。

> 参考文献
> 
> 《深入理解ElasticSearch》
> 
> 《Elasticsearch技术解析与实战》
> 
> Elasticsearch顶尖高手系列
> 
> Elasticsearch核心技术与实战
> 
> https://www.elastic.co/cn/what-is/elasticsearch