# Nginx 进程结构
这篇文章我们来看下 Nginx 的进程结构，Nginx 其实有两种进程结构：

- 单进程结构
- 多进程结构

单进程结构实际上不适用于生产环境，只适合我们做开发调试使用。因为在生产环境中我们必须保持 Nginx 足够健壮以及 Nginx 可以利用多核的一个特性，而单进程的 Nginx 是做不到这一点的，所以默认的配置中都是打开为多进程的 Nginx。

我们来看一下，多进程的 Nginx 结构中它的进程模型是怎样的。

![Nginx进程结构](https://img-blog.csdnimg.cn/20191112234545969.png)

多进程中的 Nginx 进程架构如下图所示，会有一个父进程（Master Process），它会有很多子进程（Child Processes），这些子进程会分为两类：

- worker 进程
- cache 相关的进程

## 为什么 Nginx 采用多进程结构而不是多线程结构呢？

因为 Nginx 最核心的一个目的是要保持高可用性、高可靠性，而当 Nginx 如果使用的是多线程结构的时候，因为线程之间是共享同一个地址空间的，所以当某一个第三方模块引发了一个地址空间导致的段错误时、在地址越界出现时，会导致整个 Nginx 进程全部挂掉。而当采用多进程模型时，往往不会出现这样的问题。从上图可以看到 Nginx 在做进程设计时，同样遵循了实现高可用、高可靠这样的一个目的。

比如说在 master 进程中，通常第三方模块是不会在 master 部分加入自己的功能代码的。虽然 Nginx 在设计时，允许第三方模块在 master 进程中添加自己独有的、自定义的一些方法，但是通常没有第三方模块这么做。

master 进程被设计用来的目的是做 worker 进程的管理的，也就是所有的 worker 进程是处理真正的请求的，而 master 进程负责监控每个 worker 进程是不是在正常的工作、需不需要做重新载入配置文件、需不需要做热部署。

而 cache （缓存）是在多个 worker 进程间共享的，而且缓存不仅要被 worker 进程使用，还要被 cache manager 和 cache loader进程 使用。cache manager 和 cache loader 也是为反向代理时，后端发来的动态请求做缓存所使用的，cache loader 只是用来做缓存的载入、cache manager 用来做缓存的管理。实际上每个请求处理时，使用到缓存还是由 worker 进程来进行的。

这些进程间的通讯，都是使用共享内存来解决的。可以看到cache manager 和 cache loader各有一个进程，master 进程因为是父进程，所以肯定只有一个。那么 worker 进程为什么会有很多呢？这是因为 Nginx 采用了事件驱动引擎以后，他希望每一个 worker 进程从头到尾占有一颗CPU，所以往往不止要把 worker 进程的数量配置与我们服务器上的 CPU核数一致以外，还需要把每一个worker进程与某一颗CPU核绑定在一起，这样可以更好的使用每颗CPU核上面的CPU缓存来减少缓存失效的命中率。

以上就是 Nginx 的进程结构的介绍，了解这些后更有助于我们去配置 Nginx。

刚才我们介绍了 Nginx 使用了多进程模型，由 master 作为父进程启动许多子进程，也知道了 Nginx 父子进程之间是通过信号来管理的，接下来通过一个实例给大家直观的看下父子进程以及信号之间是如何工作的。

## Nginx 的进程结构实例
首先启动 Nginx，并在 Nginx 中启动了两个 worker 进程，通过 ps 命令可以看到当前进程 PID 和父进程 PID，有一个 nginx master 进程是由 root 用户起的，进程 PID 是 2368。还有两个 worker 进程和 cache 进程，它们是由 2368 进程起来的。它们的进程 PID 分别为 8652，8653，8655。

现在我们使用 `./sbin/nginx -s reload` 命令，会把之前的 worker 进程和 cache 进程优雅的退出，然后再使用的新的配置项启动新的 worker 进程，这里我们并没有改变配置，但是我们可以看到老的三个子进程会退出，并生成新的子进程。

可以看到，之前的三个子进程，现在已经都不在了，反而由 2368 新起了 8657，8658，8660 三个子进程。

```
[root@wupx nginx]# ps -ef|grep nginx
root      2368     1  0 Sep21 ?        00:00:00 nginx: master process /usr/sbin/nginx
root      4751  4688  0 11:41 pts/0    00:00:00 grep --color=auto nginx
nginx     8652  2368  0 Nov12 ?        00:00:00 nginx: worker process
nginx     8653  2368  0 Nov12 ?        00:00:00 nginx: worker process
nginx     8655  2368  0 Nov12 ?        00:00:00 nginx: cache manager process
[root@wupx nginx]# ./sbin/nginx -s reload
[root@wupx nginx]# ps -ef|grep nginx
root      2368     1  0 Sep21 ?        00:00:00 nginx: master process /usr/sbin/nginx
root      4753  4688  0 11:43 pts/0    00:00:00 grep --color=auto nginx
nginx     8657  2368  0 Nov12 ?        00:00:00 nginx: worker process
nginx     8658  2368  0 Nov12 ?        00:00:00 nginx: worker process
nginx     8660  2368  0 Nov12 ?        00:00:00 nginx: cache manager process
```

执行命令之后可以看到 worker 的 PID 已经变化了（之前讲过 `./sbin/nginx -s reload` 跟 `kill -SIGHUP` 作用是一样的。）。

`kill -SIGTERM` 是向现有的 worker 进程发送退出的信号，对应的 worker 进程就会退出；进程在退出时，会自动向父进程 master 发送一个退出信号，master 就知道他的子进程退出了，然后新起一个 worker 进程。

```
[root@wupx nginx]# ps -ef|grep nginx
root      2368     1  0 Sep21 ?        00:00:00 nginx: master process /usr/sbin/nginx
root      4753  4688  0 11:43 pts/0    00:00:00 grep --color=auto nginx
nginx     8657  2368  0 Nov12 ?        00:00:00 nginx: worker process
nginx     8658  2368  0 Nov12 ?        00:00:00 nginx: worker process
nginx     8660  2368  0 Nov12 ?        00:00:00 nginx: cache manager process
[root@wupx nginx]# kill -SIGTERM 8658
[root@wupx nginx]# ps -ef|grep nginx
root      2368     1  0 Sep21 ?        00:00:00 nginx: master process /usr/sbin/nginx
root      4753  4688  0 11:44 pts/0    00:00:00 grep --color=auto nginx
nginx     8657  2368  0 Nov12 ?        00:00:00 nginx: worker process
nginx     8660  2368  0 Nov12 ?        00:00:00 nginx: cache manager process
nginx     8663  2368  0 Nov12 ?        00:00:00 nginx: worker process
```

通过实例演示，我们可以看到 Nginx 的进程结构以及 Nginx 使用信号的方式，其实命令行中的许多子命令就是再向 master 进程发送信号而已。

![进程模型](https://img-blog.csdnimg.cn/20191113011633243.png)