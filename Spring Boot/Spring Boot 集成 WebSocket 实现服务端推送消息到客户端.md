假设有这样一个场景：服务端的资源经常在更新，客户端需要尽量及时地了解到这些更新发生后展示给用户，如果是 HTTP 1.1 通常会开启 ajax 请求询问服务端是否有更新，通过定时器反复轮询服务端响应的资源是否有更新。

![ajax 轮询](https://img-blog.csdnimg.cn/20200627220503451.png)

在长时间不更新的情况下，反复地去询问会对服务器造成很大的压力，对网络也有很大的消耗，如果定时的时间比较大，服务端有更新的话，客户端可能需要等待定时器达到以后才能获知，这个信息也不能很及时地获取到。

而有了 WebSocket 协议，就能很好地解决这些问题，WebSocket 可以反向通知的，通常向服务端订阅一类消息，服务端发现这类消息有更新就会不停地通知客户端。

![WebSocket](https://img-blog.csdnimg.cn/20200627220503449.png)

## WebSocket 简介

WebSocket 协议是基于 TCP 的一种新的网络协议，它实现了浏览器与服务器全双工（full-duplex）通信—允许服务器主动发送信息给客户端，这样就可以实现从客户端发送消息到服务器，而服务器又可以转发消息到客户端，这样就能够实现客户端之间的交互。对于 WebSocket 的开发，Spring 也提供了良好的支持，目前很多浏览器已经实现了 WebSocket 协议，但是依旧存在着很多浏览器没有实现该协议，为了兼容那些没有实现该协议的浏览器，往往还需要通过 STOMP 协议来完成这些兼容。

下面我们在 Spring Boot 中集成 WebSocket 来实现服务端推送消息到客户端。

## Spring Boot 集成 WebSocket

首先创建一个 Spring Boot 项目，然后在 `pom.xml` 加入如下依赖集成 WebSocket：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

### 开启配置

接下来在 `config` 包下创建一个 WebSocket 配置类 `WebSocketConfiguration`，在配置类上加入注解 `@EnableWebSocket`，表明开启 WebSocket，内部实例化 ServerEndpointExporter 的 Bean，该 Bean 会自动注册 `@ServerEndpoint` 注解声明的端点，代码如下：

```
@Configuration
@EnableWebSocket
public class WebSocketConfiguration {

    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```



# 总结

本文的完整代码在 https://github.com/wupeixuan/SpringBoot-Learn 的 `websocket` 目录下。

**最好的关系就是互相成就**，大家的**在看、转发、留言**三连就是我创作的最大动力。

> 参考 
>
> https://github.com/wupeixuan/SpringBoot-Learn