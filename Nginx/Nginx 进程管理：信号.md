Nginx 是一个多进程的程序，多进程之间进行通讯可以使用共享内存、信号等。当做进程间管理的时候，通常只使用信号。

今天就来看一下 Nginx 进程管理中的信号是怎样使用的。

# Nginx 进程管理：信号

![Nginx进程管理：信号](https://img-blog.csdnimg.cn/20191129222658612.png)

从上图可以看出，能够发送和处理信号的有 master 进程、worker 进程、Nginx 命令行。

首先让我们来看下 Master 进程。

# Master 进程
因为 master 进程会启动 worker 进程，所以它管理 worker 进程的方式首先是监控 worker 进程有没有发送 CHLD 信号，因为 Linux 操作系统中规定当子进程终止的时候会向父进程发送 CHLD 信号，所以如果 worker 进程由于一些模块代码 bug 导致 worker 进程意外终止，那么 master 进程可以立刻通过 CHLD 发现这样一个事件，然后重新把 worker 进程拉起。

Master 进程还会通过接受一些信号，来管理 worker 进程。

Master 进程可以接受的信号有：

- TERM、INT：立刻停止 Nginx 进程
- QUIT：优雅停止 Nginx 进程，不会对用户立刻发送结束连接请求（比如像 TCP 中的 reset 复位请求这样的报文）
- HUP：表示重载配置文件
- USR1：表示重新打开日志文件，做日志文件的切割
- USR2：专门针对做热部署使用
- WINCH：表示优雅的退出所有 worker 进程

其中，粉色的信号 USR2 和 WINCH 只能通过 Linux 的 kill 命令行发送信号，也就是说我们需要先找到 master 进程所在的 PID，对这个 PID 发送 USR2 或者 WINCH，而其他的 4 个有对应的 Nginx 命令的。

# Worker 进程
通常是不直接对 worker 进程发送信号的，因为我们希望由 master 进程来管理 worker 进程。虽然直接对 worker 进程发送信号，也会让 worker 进程产生同样的结果，但是通常不这样做，往往是由 master 进程管理，master 进程收到信号后，会再把信号发送给 worker 进程。


# Nginx 命令行
Nginx 在启动以后，Nginx会把他的 PID 放到一个文件中。默认是记录在 Nginx安装目录的 /logs/nginx.pid 文件中，记录了 Nginx 的 master 进程的 PID。

当我们再次使用 nginx -s 这样的命令行的时候，那么 nginx 的工具命令行就会去读取PID文件中的 master 进程的 PID，向这个 PID 发送同样的 HUP、USR1、TERM、QUIT 这样的信号，而这样的命令对应着命令 reload、reopen、stop、quit，所以调用 nginx 命令行和直接用 kill 发送信号的效果是一样的。

# 总结
这篇文章主要介绍了 Nginx 进程管理中信号的使用，主要涉及到 Master 进程、Worker 进程和 Nginx 命令行。我们可能会看到网络上有很多管理日志文件或者管理热升级的脚本，这些脚本中有时候是用 kill 直接发送信号，有时候是调用 nginx -s 这样一个命令行，很多人之前可能还会很困惑，那么我相信大家现在应该很了解这样一个过程怎样发生的了。