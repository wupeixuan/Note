Eureka组件分为两部分： Eureka Server 和 Eureka Client

在启动程序中分别添加注解：

@EnableEurekaServer  @EnableEurekaClient

## Eureka功能
心跳检测、健康检查、负载均衡等

## Eureka高可用

启动三个Eureka Server，端口分别为 8761，8762，8763，每一个Eureka注册到另外两个Eureka上。

在 VM options中分别填写

```
-Dserver.port=8761

-Dserver.port=8762

-Dserver.port=8763
```



第一个Eureka Server的配置文件application.yml中添加
```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8762/eureka/,http://localhost:8763/eureka/
```

配置好后，分别启动三个 Eureka Server

在eureka工程路径下执行

```
mvn clean package
```

再进入target目录，执行

```
java -jar eureka-0.0.1-SNAPSHOT.jar
```


Eureka Client 注册到三个 Eureka Server 上，配置文件application.yml中添加

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/,http://localhost:8762/eureka/,http://localhost:8763/eureka/
```

## 服务发现的两种方式
- 客户端发现：Eureka
- 服务端发现：Nginx，Zookeeper，Kubernetes