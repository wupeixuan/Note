## 下载Kafka
1.下载Kafka:http://mirror.bit.edu.cn/apache/kafka/2.3.0/kafka_2.12-2.3.0.tgz

2.解压后复制Kafka文件夹,分别命名为kafka1、kafka2、kafka3 

## 修改配置文件
1. 修改config文件夹下的server.properties ，其中的brokerId是惟一的,集群中kafka服务器配置的brokerId不能相同，相当于zookeeper的myid
2. zookeeper.connect=127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183/kafka

> 说明: 
> 这个是zookeeper集群的服务器端口号, /kafka是在zookeeper挂载的文件夹,要自己创建zookeeper客户端命令 create /kafka

## Kafka操作

创建主题

`kafka-topics.bat –create –zookeeper 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183/kafka –replication-factor 1 –partitions 1 –topic test `

> 指令说明: 
> –create 指定创建topic动作 
> –zookeeper 指定kafka连接zk的连接url，该值和server.properties文件中的配置项{zookeeper.connect}一样 
> –replication-factor：指定每个分区的复制因子个数，默认1个 
> –partitions：指定当前创建的kafka分区数量，默认为1个 
> –topic:设置主题名字 


查看主题状态

`kafka-topics.bat –describe –zookeeper 127.0.0.1:2181,127.0.0.1:2182,127.0.0.1:2183/kafka –topic test` 

> 结果信息字段含义: 
> 1 Partition： 分区 
> 2 Leader ： 负责读写指定分区的节点 
> 3 Replicas ： 复制该分区log的节点列表 
> 4 Isr ： “in-sync” replicas，当前活跃的副本列表（是一个子集），并且可能成为Leader


kafka生产者生产消息

`kafka-console-producer.bat –broker-list 127.0.0.1:9092 –topic test`

消费者接受消息

`kafka-console-consumer.bat --bootstrap-server 127.0.0.1:9092 --topic test --from-beginning`