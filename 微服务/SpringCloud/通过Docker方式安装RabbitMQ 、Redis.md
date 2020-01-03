## RabbitMQ
首先进入 RabbitMQ 官网

找到对应的版本

这里是最新版本 3.7.17-management，找到你需要安装的版本， -management 表示有管理界面的，可以浏览器访问。

接来下docker安装，我这里装的 3.7.17-management：

`docker run -d --hostname my-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3.7.17-management`

`docker ps`

列出所有在运行的容器信息



用户名/密码： guest/guest

## Redis

`docker run -d -p 6379:6379 redis:4.0.8`