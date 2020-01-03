## Nginx的启动
指定配置文件的方式启动

`nginx -c /etc/nginx/nginx.conf`

对于yum安装的nginx，使用systemctl命令启动
`systemctl start nginx`

## Nginx的停止
查询Nginx的进程

`ps -ef|grep nginx`

从容停止

`kill -QUIT Nginx主进程号`

快速停止Nginx

`kill -TERM Nginx主进程号`

强制停止所有nginx进程

`pkill -9 nginx`

对于yum安装的nginx，使用systemctl命令停止

`systemctl stop nginx`

## Nginx的平滑重启

在修改了nginx配置文件后，在重启nginx之前，需要确认nginx配置文件的语法是否正确，可执行以下命令检测

`nginx -t -c /etc/nginx/nginx.conf`
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
当显示测试成功时，就可以平滑重启了

`nginx -s reload`或者`kill -HUP Nginx主进程号`

对于yum安装的nginx，使用systemctl命令重启

`systemctl restart nginx`

