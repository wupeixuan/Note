在my.ini中添加：
```
log-bin=mysql-bin
log-bin="D:\Software/MySQL/MySQL Server 5.5/Data/log"
expire_logs_days=10
max_binlog_size=100M
```

重启MYSQL服务

查询配置的日志是否开启

`show VARIABLES LIKE '%log_%';`


查看BinLog日志

`show binlog events in 'log.000001';`

利用BinLog恢复数据库

`mysqlbinlog --stop-datetime="2017-04-17 22:02:00" "D:\Software\MySQL\MySQL Server 5.5\Data\log.000001" |mysql -u root –p`

ErrorLog配置

```
log-error=[path/[file_name]]
show variables LIKE 'log_error';
```

慢日志查询配置

```
log-slow-queries[=path/[filename]] 
long_query_time=10
```

慢日志查询

```
show variables like '%slow%';
select sleep(10);
```

主从复制

Master配置
```
[mysqld] 
log-bin="D:/MYSQLDataBase/binlog" expire_logs_days=10 
max_binlog_size=100M 
server-id=1 
binlog-do-db=test 
binlog-ignore-db=mysql
```

Slave配置
```
[mysql]
default-character-set=utf8
log_bin="C:/MYSQLLOG/binlog"
expire_logs_days=10
max_binlog_size=100M
server-id=2
```


语法：
```
Stop Slave;
change master to master_host='192.168.1.100', 
master_user='repl',
 master_password='123',
 master_log_file='binlog.000004', 
master_log_pos=107;
Start slave;
```

各个参数所代表的具体含义如下：
```
master_host：表示实现复制的主机ip地址
master_user：表示实现复制的登录远程主机的用户
master_password：表示实现复制的登录远程主机的密码
master_log_file：表示实现复制的binlog日志文件
master_log_pos：表示实现复制的binlog日志文件的偏移量
```
```
show master status
show slave status
```
