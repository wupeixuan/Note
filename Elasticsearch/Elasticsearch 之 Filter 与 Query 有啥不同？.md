今天来了解下 Elasticsearch（以下简称 ES） 中的 Query 和 Filter。

在 ES 中，提供了 Query 和 Filter 两种搜索：

- Query Context：会对搜索进行相关性算分
- Filter Context：不需要相关性算分，能够利用缓存来获得更好的性能

举一个栗子，比如需要搜索一场电影，包含以下信息：

评论中包含了烧脑，评分高于 8 分，同时上映时间在 2010 到 2020 之间。

所以这个搜索包括了三个判断逻辑，针对三个不同的字段进行查询，如果需要满足这样的查询需求，在 ES 当中提供了 bool 查询，一个 bool 查询可以包含一个或多个查询字句，支持以下四种查询：

- must：必须匹配，贡献算分
- should：选择性匹配，贡献算分
- must_not：查询字句，必须不能匹配
- filter：必须匹配，不贡献算分

![](https://img-blog.csdnimg.cn/20200717160959843.png)

上图是一个 bool 查询，是对用户（`user`）进行搜索，城市必须是北京（`beijing`） ，性别必须是男（`man`），这个采用的是 filter，说明这个对算分是不会产生影响的，`must_not` 是一个 range 的查询：年龄大于等于 35 岁；should 里是一个数组，说明这个 should 中可以写多个条件，只要用户的名字是这两个中的一个就是满足条件的。

其实，bool 查询的子查询可以任意顺序出现，并且可以嵌套多个查询。

另外，should 的使用分两种情况：

- bool 查询中只包含 should，不包含 must 查询
- bool 查询中同时包含 should 和 must 查询

下面让我们来看看这两种情况有何不同？

如果在 bool 查询中没有 must 子句，should 中必须至少满足一条查询（可以通过 `minimum_should_match` 来设置满足条件的个数或者百分比）。

同时包含 should 和 must 时，文档不必满足 should 中的条件，但是如果满足条件，会增加相关性算分。

### Filter Context

上面说到了 `filter` 和 `must_not` 是不会影响算分的，通过查询结果中可以看到 `_score` 都是 0。

![](https://img-blog.csdnimg.cn/20200718162655500.png)

### Query Context

采用 should 查询，会进行算分处理，结果如下图所示：

![](https://img-blog.csdnimg.cn/20200718163309760.png)

同时，查询语句的结构，也会对相关度算分产生影响：

- 同一层级的查询字段，权重是相同的
- 通过嵌套 bool 查询，可以改变对算分的影响

### Boost & Boosting Query

相关度还可以通过对某个字段设置 `boost` 的值来进行控制：

- 当 boost > 1 时，打分的相关度相对性提升
- 当 0 < boost < 1 时，打分的权重相对性降低
- 当 boost < 0 时，贡献负分

或者使用 ES 提供的 Boosting Query 进行查询：

首先插入几条数据用于测试：

```
POST /product/_bulk
{ "index": { "_id": 1 }}
{ "content":"Apple Mac" }
{ "index": { "_id": 2 }}
{ "content":"Apple iPad" }
{ "index": { "_id": 3 }}
{ "content":"Apple Juice" }
```

如下图所示，左边就是一个 Boosting Query，positive 查询意思是如果 `content` 中包含 `Apple` 会按照原始的相关性分数进行打分，negative 查询则是满足 positive 查询同时满足 negative 查询（`content` 中包含 `Juice`）的会按照原始的相关性分数乘以 `negative_boost` 进行打分，`negative_boost` 是用于降低与 negative 匹配文档的相关性算分的。

![](https://img-blog.csdnimg.cn/20200718174314172.png)

如右图所示，这个的查询结果为三条数据，可以发现 `Apple Mac` 和 `Apple iPad` 的相关性算分相同，都排在前面，而 `Apple Juice` 的相关性算分是其他两个的 0.1 倍，排在最后。

用一个表格来总结下 Query Context 和 Filter Context 的区别：

| Context Type | 含义                                                       | 使用方式                                                 |
|--------------|------------------------------------------------------------|----------------------------------------------------------|
| Query        | 查找与查询语句最匹配的文档，对所有文档进行相关性算分并排序 | query；bool 中的 must 和 should                          |
| Filter       | 查找与查询语句相匹配的文档                                 | bool 中的 filter 和 must_not；constant_score 中的 filter |

filter 不需要计算相关性算分，不需要按照相关分数进行排序，同时还有内置的自动 cache 最常使用的 filter 的数据，而 query 相反，需要计算相关性算分，按照分数进行排序，而且无法 cache 结果，因此在某些不需要相关性算分的查询场景，尽量使用 Filter Context 来让查询更加高效。

下图为 eBay 对于 Filter Context 和 Query Context 的性能比较：

![Filter Context VS Query Context](https://img-blog.csdnimg.cn/20200718185429456.png)

那么 filter 的 cache 是怎么做的呢？

ES 会构建一个文档匹配过滤器的位集 bitset（用来标识一个文档对一个 filter 条件是否匹配，如果匹配就是 1，不匹配就是 0），下次再有这个 filter 条件过来的时候就不用重新扫描倒排索引，反复生成 bitset，可以大幅度提升性能，另外当添加或更新文档时，这个 filter 的位集 bitset 也会更新。

# 总结

当用户输入多个条件进行查询的时候，可以使用 bool 查询，在 bool 查询中，`filter` 和 `must_not` 属于 Filter Context，不会对算分结果产生影响；`must` 和 `should` 属于 Query Context，会对结果算分产生影响。

在 bool 查询中，查询结构是对相关性算分有影响的，可以通过嵌套的方式修改不同字段在查询中的权重以及直接通过指定字段的 boost 值来控制在搜索中的权重，另外使用 Boosting Query 可以提升搜索的精准性，同时也可以将更多的搜索显示在结果中。

**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考
> 
> Elastic Stack从入门到实践
> 
> Elasticsearch核心技术与实战
>
> Elasticsearch顶尖高手系列-快速入门篇
> 
> https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html
>
> https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-boosting-query.html