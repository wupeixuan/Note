由于 RabbitMQ 是基于 Erlang 语言开发，所以在安装 RabbitMQ 之前，需要先安装 Erlang。

# 安装 Erlang 环境
1. 安装依赖
```
yum install gcc glibc-devel make ncurses-devel openssl-devel xmlto
```

2. 安装 Erlang
```
wget -c http://erlang.org/download/otp_src_20.2.tar.gz
# 解压
tar -zxvf otp_src_20.2.tar.gz
cd otp_src_20.2/
# 编译安装
./configure --prefix=/data/soft/erlang
make && make install
# 配置环境变量
vim /etc/profile
ER_LANG=/data/soft/erlang
PATH=$PATH:$ER_LANG/bin:
# 更新配置文件
source /etc/profile
```

测试是否安装成功:`erl`

出现如下结果，就是安装成功了
```
Erlang/OTP 22 [erts-10.5] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:1] [hipe]

Eshell V10.5  (abort with ^G)
```

输入 halt().  退出控制台。

# 安装 RabbitMQ
```
cd /data/soft/
wget -c http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.15/rabbitmq-server-generic-unix-3.6.15.tar.xz
# 解压
xz -d rabbitmq-server-generic-unix-3.6.15.tar.xz 
tar -xvf rabbitmq-server-generic-unix-3.6.15.tar
# 配置环境变量
vim /etc/profile
RABBIT_MQ=/data/soft/rabbitmq
PATH=$PATH:$ER_LANG/bin:$RABBIT_MQ/sbin:
# 更新配置文件
source /etc/profile
```

RabbitMQ 的基本操作：
```
# 启动
rabbitmq-server -detached
# 关闭
rabbitmqctl stop
# 查看状态
rabbitmqctl status
```

配置 RabbitMQ 网页管理插件：`rabbitmq-plugins enable rabbitmq_management`

开启 RabbitMQ 远程访问
- 添加用户:rabbitmqctl add_user wupx 123　　//wupx 是用户名， 123 是用户密码
- 添加权限:rabbitmqctl set_permissions -p "/" wupx ".*" ".*" ".*"
- 修改用户角色:rabbitmqctl set_user_tags wupx administrator

打开 http://192.168.81.66:15672 ，输入用户名密码就可以访问了。