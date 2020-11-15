# 报错情况

在 Linux 环境下启动 Elasticsearch 的时候，会报错：

```
[root@dev-es bin]# ./elasticsearch
[2020-08-07T12:11:07,538][WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [xmgl-dev-es-3-90] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:163) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:150) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) ~[elasticsearch-cli-6.6.2.jar:6.6.2]
        at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:116) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:93) ~[elasticsearch-6.6.2.jar:6.6.2]
Caused by: java.lang.RuntimeException: can not run elasticsearch as root
        at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:103) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:170) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:333) ~[elasticsearch-6.6.2.jar:6.6.2]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:159) ~[elasticsearch-6.6.2.jar:6.6.2]
        ... 6 more
```

因为在 Elasticsearch 5.X 之后为了安全起见，不能用 root 用户来启动，必须用非 root 用户启动

# 解决方法

1. 创建一个新用户 `useradd es`
2. 给新用户授权 `chown -R es.es elasticsearch-6.6.0`
3. 进入到 bin 目录，使用命令 `./elasticsearch -d`  启动

