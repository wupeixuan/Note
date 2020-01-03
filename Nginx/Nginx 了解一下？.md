这篇文章主要简单的介绍下 Nginx 的相关知识，主要包括以下几部分内容：
1. Nginx 适用于哪些场景？
2. 为什么会出现 Nginx？
3. Nginx 优点
4. Nginx 的编译与配置

# Nginx 适用于哪些场景？
![Nginx 适用场景](https://img2018.cnblogs.com/blog/1356806/201909/1356806-20190929130523078-624143551.png)

如图所示，一个请求会先经过 Nginx 到达应用服务层，然后再去访问数据层（比如 Redis、MySQL 等），提供基本的数据功能。我们的应用服务因为要求开发效率是非常高的，所以它的运行效率是很低的，它的 qps、tps或者并发都是受限的，所以我们需要把很多这样的应用服务组成集群，向用户提供高可用服务。而一旦很多服务构成集群的时候，我们需要 Nginx 具备反向代理功能，可以把动态请求传递给应用服务。

而当应用服务构成集群，一定会带来两个需求：

1. 需要动态的扩容
2. 有些服务出现问题的时候我们需要做容灾

这样反向代理必须具备负载均衡功能。

其次在这样的一个链路中， Nginx 是处在企业内网的一个边缘节点，随着网络链路的增长，用户体验到的时延会增加，所以需要把用户看起来不变的或者在一段时间内看起来不变的动态内容缓存在 Nginx 部分，由 Nginx 直接向用户提供访问，这样用户时延就会减少很多。所以反向代理延伸出另外一个功能就是缓存，来减少用户访问的时延。

像很多 css、js、img 静态资源，是没有必要通过应用服务来访问的，只需要本地文件系统上放置的静态资源，直接由 Nginx 提供访问就可以了。这是 Nginx 的静态资源服务。

应用服务本身的性能存在很多问题，像数据库服务比应用服务好的多，因为业务场景比较简单，并发性能和tps都要远高于应用服务，所以延伸出第三个应用场景：由 Nginx 直接去访问数据库、Redis，利用 Nginx 强大的并发性能实现如 web防火墙 复杂的一些业务功能。这就需要api服务有很强的业务处理功能，所以像 OpenResty、 Nginx 集成的 JavaScript，应用 JavaScript、lua 这样的语言功能和它们语言自带的一些工具库来提供完整的 API服务。

# 为什么会出现 Nginx？
伴随着互联网的快速普及、以及全球化和物联网的快速发展，导致互联网的数据量快速增长。

CPU 核数从当初的单核发展到 16 核，甚至 32 核，但是由于操作系统和大量的软件没有做好服务于多核架构的准备，致使服务的性能通常不会有成倍的提升。

Apache 的架构模型一个进程同一时间只会处理一个链接一个请求，处理完以后才会处理下一个请求。它实际上在使用操作系统的进程间切换的特性，因为操作系统微观上只有有限的 CPU，但是操作系统被设计为同时服务数百甚至上千的进程，而 Apache 一个进程只能服务于一个链接，这样的模式会导致当 Apache 需要面对几十万、几百万链接的时候，它没有办法去开几十万、几百万的进程；而进程间切换的代价成本又太高了，当并发的连接数越多，这种无谓的进程间切换引发的性能消耗也就越大，而 Nginx 是专门为了这样的应用场景而生的，Nginx 可以处理数百万甚至上千万的并发链接。

# Nginx 优点
一、高并发，高性能

只要我们对每个链接使用的内存足够少就能实现高并发；既要达到高并发又要达到高性能，往往需要很好的设计。

比如现在的主流云服务器，nginx 在 32 核 64G 的配置中可以轻松达到数千万的并发链接；如果是处理简单的静态资源请求，nginx 可以达到 100w 的 RPS 。

> RPS（Requests Per Second）为每秒能处理的请求数目，等效于 QPS（Queries Per Second），也就是每秒能处理查询数目。是一台服务器每秒能够相应的查询次数，是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准。

二、可扩展性好

可扩展性主要体现在模块化设计；模块化设计非常稳定，使得 Nginx 的生态圈、第三方模块非常丰富。甚至于有 Tengine、OpenResty 这样的第三方插件在他的基础之上又生成了新的生态圈。丰富的生态圈和第三方模块为 Nginx 的丰富功能提供了保证。

三、高可靠性

高可靠性指的是 Nginx 可以服务器上持续不间断的运行数年，而很多web服务器往往运行几周or几个月，就需要进行一次重启。对于 Nginx 这样一个高并发、高性能的反向代理服务器而言，往往运行在企业内网的边缘节点上，这个时候如果我们企业想提供4个9、5个9、甚至更高的高可用性时，对于 Nginx 持续运行能够宕机的时间一年可能只能以秒来计，所以在这样一个角色中，Nginx 的高可靠性给我们提供了非常好的保证。

四、热部署

热部署是指在不停止服务的情况下升级Nginx。这个功能对于 Nginx 来说非常重要，因为在服务器上跑了数百万的并发链接，如果是普通的服务器，我们只能 kill 掉进程再重启的方式进行升级操作。但是对于 Nginx 而言，因为直接 kill 掉 nginx 进程会给所有的已经建立链接的客户端一个很不好的体验。

五、BSD 许可证

BSD许可证是指 Nginx 不只是开源的、免费的，而且我们可以在有定制需求的场景下，去修改 Nginx 的源码，再运行在我们的商业场景下且属于合法的。

# Nginx 组成
Nginx 主要由以下 4 部分组成：
- Nginx 二进制可执行文件：由各模块源码编译出的一个文件
- Nginx.conf 配置文件：控制 Nginx 行为
- access.log 访问日志：记录每一条 http 请求信息
- error.log 错误日志：定位问题

接下来，我们就要动手去编译 Nginx 了。
# 编译 Nginx
```
# 下载
wget http://nginx.org/download/nginx-1.14.0.tar.gz
# 解压
tar -xzvf nginx-1.14.0.tar.gz
cd nginx-1.14.0
# 配置
./configure --prefix=/usr/local/nginx
# 编译
make
# 安装
make install
```

在 configure 过程中可能遇到的问题：

> ./configure: error: the HTTP rewrite module requires the PCRE library. You can either disable the module by using --without-http_rewrite_module option, or install the PCRE library into the system, or build the PCRE library statically from the source with nginx by using --with-pcre= option.

> ./configure: error: the HTTP gzip module requires the zlib library. You can either disable the module by using --without-http_gzip_module option, or install the zlib library into the system, or build the zlib library statically from the source with nginx by using --with-zlib= option.


出错的原因是 Nginx 模块需要依赖一些 lib 库，解决办法如下：
```
安装 pcre-devel 和 zlib-devel 依赖库：yum -y install pcre-devel zlib-devel
```

# Nginx 配置
## Nginx 配置语法
![Nginx配置语法](https://img-blog.csdnimg.cn/20191101005045207.png)

## Nginx 配置参数
### 配置参数：时间的单位
![时间的单位](https://img-blog.csdnimg.cn/20191101001834675.png)

### 配置参数：空间的单位
![空间的单位](https://img-blog.csdnimg.cn/20191101001928692.png)

### http 配置的指令块
![http配置的指令块](https://img-blog.csdnimg.cn/20191101001511883.png)
- http：表示里面所有的指令都是由 http 模块去解析去执行的
- server：解析对应的域名or一组域名
- location：url 表达式
- upstream：表示上游服务，需要与企业内网服务直连的时候，可以定义一个 upstream

### 示例
![示例](https://img-blog.csdnimg.cn/20191101002431491.png)

示例中的所有指令都是由 Nginx 中的 http 模块去执行的，其中 `server 127.0.0.1:8000` 为需要解析的域名，location 后面跟的为对应的匹配规则，`expires 3m`表示 3 分钟后 cache 刷新，`zone=one:10m`表示开辟了一个 10m 大小的共享内存空间，给不同的 worker 去使用。

# 总结
这篇文章主要介绍了 Nginx 出现的原因和使用场景，并分析 Nginx 的优点，最后动手去编译属于自己的 Nginx，并进行简单配置。

> 参考
> 
> http://nginx.org/en/docs/
>
> 《Nginx核心知识100讲》