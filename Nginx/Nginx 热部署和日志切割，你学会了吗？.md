上篇文章，我们已经安装并配置好了 Nginx，接下来我们就需要操作 Nginx 命令行了，这篇文章主要讲解 Nginx 命令行相关知识，并通过日常开发中遇到的热部署、切割日志文件案例来熟悉 Nginx 命令行操作。

# Nginx 命令行
1. 格式：nginx -s stop
2. 帮助：-? -h
3. 使用指定的配置文件：-c
4. 指定配置指令：-g （用途是覆盖配置文件中的指令）
5. 指定运行目录：-p
6. 发送信号：-s（立刻停止服务：stop，优雅的停止服务：quit，重新配置文件：reload，重新开始记录日志文件：reopen）
7. 测试配置文件是否有语法错误：-t   -T
8. 打印 nginx 的版本信息、编译信息等：-v    -V

Nginx 命令和大部分的 Linux 的命令很相似，都是 nginx 加基本指令，再加指令相关的参数。默认情况下 nginx 会去寻找之前执行 configure 命令时指定位置的配置文件，但是可以通过 -c 来指定配置文件，并且可以通过 -g 来指定配置指令。

nginx 去操作运行中进程的方法一般是通过发送信号，可以通过 linux 通用的 kill 命令，也可以用 nginx 的 -s 命令来发送信号。

接下来，让我们通过几个栗子来熟悉 Nginx 的命令行操作。

# 重载配置文件
配置文件默认是在安装目录的 conf 文件下，文件名为 nginx.conf，我们可以打开看一下：
```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;

    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

假如我们需要开启 gzip 压缩，我们可以把它前面的注释去掉，当我们在修改完 nginx 配置文件后，我们可以通过 nginx 的命令 `./nginx -s reload` 重启 nginx 服务。

# Nginx 热部署
当从老版本替换为新版本的 nginx 的时候，如果不热部署的话，会需要取消 nginx 服务并重启服务才能替换成功，这样的话会使正在访问的用户在断开连接，所以为了在不影响用户的体验下进行版本升级，就需要热部署来升级版本。

接下来，让我们一起进行一次热部署吧。

因为进行升级主要是更换二进制文件，所以在升级前先备份旧的二进制文件。
```
# 备份旧版本的 nginx 二进制文件
mv /usr/local/nginx/sbin/nginx  /usr/local/nginx/sbin/nginx.old
```

然后下载最新版本的 nginx，解压后进行编译，再把编译好的最新版本的 nginx 二进制文件拷贝到安装目录下的 sbin 目录下。

```
# 到官网下载最新版本的 nginx
wget http://nginx.org/download/nginx-1.17.2.tar.gz
# 解压
tar -xzvf nginx-1.17.2.tar.gz
cd nginx-1.17.2
./configure --prefix=/usr/local/nginx
# 编译
make
# 替换旧的 nginx 的执行程序
cp -r /usr/local/nginx-1.16.1/objs/nginx /usr/local/nginx/sbin/ -f
```

通过 `ps -ef | grep nginx` 来查看 nginx 运行状况：

```
[root@wupx sbin]# ps -ef | grep nginx 
root      1752     1  0 20:39 ?        00:00:00 nginx: master process ./sbin/nginx
nobody    1783  1752  0 20:41 ?        00:00:00 nginx: worker process
root      1787     1  0 20:41 ?        00:00:00 wget http://nginx.org/download/nginx-1.17.2.tar.gz
root      4357  1708  0 21:00 pts/2    00:00:00 grep --color=auto nginx
```

可以看到目前启动的 nginx 的 PID 为 1752，下面需要给正在运行的 nginx 的 master 进程发送信号，告诉它我们要进行热部署了。

```
# 发送 USR2 信号给旧版本主进程号,使 nginx 的旧版本停止接收请求，用 nginx 新版本接替
kill -USR2 1752
```

再通过 `ps -ef | grep nginx` 来查看 nginx 运行状况：
```
[root@wupx sbin]# ps -ef | grep nginx 
root      1752     1  0 20:39 ?        00:00:00 nginx: master process ./sbin/nginx
nobody    1783  1752  0 20:41 ?        00:00:00 nginx: worker process
root      1787     1  0 20:41 ?        00:00:00 wget http://nginx.org/download/nginx-1.17.2.tar.gz
root      4391  1752  0 21:02 ?        00:00:00 nginx: master process ./sbin/nginx
nobody    4392  4391  0 21:02 ?        00:00:00 nginx: worker process
root      4394  1708  0 21:07 pts/2    00:00:00 grep --color=auto nginx
```
这个时候我们需要给老的 nginx 发送信号，告诉老的 nginx 请优雅的关闭所有的 worker 进程。

```
# 发送 WINCH 信号到旧的主进程，它会通知旧的 worker 进程优雅的关闭，然后退出
kill -WINCH 1752
```

重新在查看 nginx 状态：
```
[root@wupx sbin]# ps -ef | grep nginx 
root      1752     1  0 20:39 ?        00:00:00 nginx: master process ./sbin/nginx
root      1787     1  0 20:41 ?        00:00:00 wget http://nginx.org/download/nginx-1.17.2.tar.gz
root      4391  1752  0 21:02 ?        00:00:00 nginx: master process ./sbin/nginx
nobody    4392  4391  0 21:02 ?        00:00:00 nginx: worker process
root      4402  1708  0 21:08 pts/2    00:00:00 grep --color=auto nginx
```

也可以发现老的 nginx maser 进程还存在，它的意义是：如果存在问题，需要退回到老版本中，我们可以给它发送 reload 命令，让他重新把 worker 进程拉起来、把新版本关掉。保留在这里方便我们做版本回退。

如果要退出保留的 master 进程，可以通过 `kill -QUIT` 命令来完成:

```
# 发送 QUIT 信号到旧的主进程，它会退出保留的 master 进程
kill -QUIT 1752
```
执行完后，1752 进程退出，通过 netstat lntup 可以看到 80 端口已经被 4391 进程监听了（新版本 nginx 的进程）。

到此为止，我们就完成了 nginx 的热部署。

# 日志切割
为了避免日志文件过大不方便查看，因此需要对日志切割。首先将原先的日志进行备份：

```
# 备份原日志
mv error.log old_error.log
```
查看日志大小：
```
[root@wupx logs]# ll
total 20
-rw-r--r-- 1 root root 6789 Nov  6 22:28 access.log
-rw-r--r-- 1 root root    5 Nov  6 22:16 nginx.pid
-rw-r--r-- 1 root root 7831 Nov  6 22:28 old_error.log
```

接下来进行日志切割：
```
# 日志切割
/usr/local/nginx/sbin/nginx -s reopen
```

再次查看：
```
[root@wupx logs]# ll
total 24
-rw-r--r-- 1 nobody root 6789 Nov  6 22:28 access.log
-rw-r--r-- 1 nobody root   60 Nov  6 22:30 error.log
-rw-r--r-- 1 root   root    5 Nov  6 22:16 nginx.pid
-rw-r--r-- 1 root   root 7831 Nov  6 22:28 old_error.log
```
经过上面的操作，我们就完成了日志的切割，以上操作只是为了了解日志切割的操作流程，不建议直接生产这么使用。推荐先写成一个 shell 脚本，通过 shell 脚本去定时执行。

示例脚本：
```
#!/bin/bash
LOGS_PATH=/usr/local/nginx/logs/history
CUR_LOGS_PATH=/usr/local/nginx/logs
YESTERDAY=$(date -d "yesterday" +%Y-%m-%d)
mv ${CUR_LOGS_PATH}/access.log ${LOGS_PATH}/old_access_${YESTERDAY}.log
mv ${CUR_LOGS_PATH}/error.log ${LOGS_PATH}/old_error_${YESTERDAY}.log
## 向 NGINX 主进程发送 USR1 信号。USR1 信号是重新打开日志文件
kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid)
```

# 总结
这篇文章主要介绍了 Nginx 命令行相关知识，并介绍了重载配置文件、Nginx 热部署、日志切割等操作，还是需要多实践操作，实践出真知。

![Nginx 命令行](https://img-blog.csdnimg.cn/20191102012406881.png)