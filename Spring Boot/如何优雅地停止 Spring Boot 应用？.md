首先来介绍下什么是优雅地停止，简而言之，就是对应用进程发送停止指令之后，能保证**正在执行的业务操作不受影响，可以继续完成已有请求的处理，但是停止接受新请求**。

在 Spring Boot 2.3 中增加了新特性**优雅停止**，目前 Spring Boot 内置的四个嵌入式 Web 服务器（`Jetty、Reactor Netty、Tomcat 和 Undertow`）以及反应式和基于 Servlet 的 Web 应用程序都支持优雅停止。

下面，我们先用新版本尝试下：

## Spring Boot 2.3 优雅停止

首先创建一个 Spring Boot 的 Web 项目，版本选择 `2.3.0.RELEASE`，Spring Boot `2.3.0.RELEASE` 版本内置的 Tomcat 为 `9.0.35`。

然后需要在 `application.yml` 中添加一些配置来启用优雅停止的功能：

```
# 开启优雅停止 Web 容器，默认为 IMMEDIATE：立即停止
server:
  shutdown: graceful

# 最大等待时间
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

其中，平滑关闭内置的 Web 容器（以 Tomcat 为例）的入口代码在 `org.springframework.boot.web.embedded.tomcat` 的 `GracefulShutdown` 里，大概逻辑就是先停止外部的所有新请求，然后再处理关闭前收到的请求，有兴趣的可以自己去看下。

内嵌的 Tomcat 容器平滑关闭的配置已经完成了，那么如何优雅关闭 Spring 容器了，就需要 Actuator 来实现 Spring 容器的关闭了。

然后加入 `actuator` 依赖，依赖如下所示：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

然后接着再添加一些配置来暴露 actuator 的 shutdown 接口：

```
# 暴露 shutdown 接口
management:
  endpoint:
    shutdown:
      enabled: true
  endpoints:
    web:
      exposure:
        include: shutdown
```

其中通过 Actuator 关闭 Spring 容器的入口代码在 `org.springframework.boot.actuate.context` 包下 `ShutdownEndpoint` 类中，主要的就是执行 `doClose()` 方法关闭并销毁 `applicationContext`，有兴趣的可以自己去看下。

配置搞定后，然后在 `controller` 包下创建一个 `WorkController` 类，并有一个 `work` 方法，用来模拟复杂业务耗时处理流程，具体代码如下：

```
@RestController
public class WorkController {

    @GetMapping("/work")
    public String work() throws InterruptedException {
        // 模拟复杂业务耗时处理流程
        Thread.sleep(10 * 1000L);
        return "success";
    }
}
```

然后，我们启动项目，先用 Postman 请求 `http://localhost:8080/work` 处理业务：

![](https://img-blog.csdnimg.cn/20200520230257966.png)

然后在这个时候，调用 `http://localhost:8080/actuator/shutdown` 就可以执行优雅地停止，返回结果如下：

```
{
    "message": "Shutting down, bye..."
}
```

如果在这个时候，发起新的请求 `http://localhost:8080/work`，会没有反应：

![](https://img-blog.csdnimg.cn/20200520230533642.png)

再回头看第一个请求，返回了结果：`success`。

其中有几条服务日志如下：

```
2020-05-20 23:05:15.163  INFO 102724 --- [     Thread-253] o.s.b.w.e.tomcat.GracefulShutdown        : Commencing graceful shutdown. Waiting for active requests to complete
2020-05-20 23:05:15.287  INFO 102724 --- [tomcat-shutdown] o.s.b.w.e.tomcat.GracefulShutdown        : Graceful shutdown complete
2020-05-20 23:05:15.295  INFO 102724 --- [     Thread-253] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
```

从日志中也可以看出来，当调用 `shutdown` 接口的时候，会先等待请求处理完毕后再优雅地停止。

到此为止，Spring Boot 2.3 的优雅关闭就讲解完了，是不是很简单呢？如果是在之前不支持优雅关闭的版本如何去做呢？

## Spring Boot 旧版本优雅停止

在这里介绍 GitHub 上 issue 里 Spring Boot 开发者提供的一种方案：

选取的 Spring Boot 版本为 `2.2.6.RELEASE`，首先要实现 `TomcatConnectorCustomizer` 接口，该接口是自定义 `Connector` 的回调接口：

```
@FunctionalInterface
public interface TomcatConnectorCustomizer {

	void customize(Connector connector);
}
```

除了定制 `Connector` 的行为，还要实现 `ApplicationListener<ContextClosedEvent>` 接口，因为要监听 Spring 容器的关闭事件，即当前的 ApplicationContext 执行 `close()` 方法，这样我们就可以在请求处理完毕后进行 Tomcat 线程池的关闭，具体的实现代码如下：

```
@Bean
public GracefulShutdown gracefulShutdown() {
    return new GracefulShutdown();
}

private static class GracefulShutdown implements TomcatConnectorCustomizer, ApplicationListener<ContextClosedEvent> {
    private static final Logger log = LoggerFactory.getLogger(GracefulShutdown.class);

    private volatile Connector connector;

    @Override
    public void customize(Connector connector) {
        this.connector = connector;
    }

    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        this.connector.pause();
        Executor executor = this.connector.getProtocolHandler().getExecutor();
        if (executor instanceof ThreadPoolExecutor) {
            try {
                ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executor;
                threadPoolExecutor.shutdown();
                if (!threadPoolExecutor.awaitTermination(30, TimeUnit.SECONDS)) {
                    log.warn("Tomcat thread pool did not shut down gracefully within 30 seconds. Proceeding with forceful shutdown");
                }
            } catch (InterruptedException ex) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

有了定制的 `Connector` 回调，还需要在启动过程中添加到内嵌的 Tomcat 容器中，然后等待监听到关闭指令时执行，`addConnectorCustomizers` 方法可以把定制的 `Connector` 行为添加到内嵌的 Tomcat 中，具体代码如下：

```
@Bean
public ConfigurableServletWebServerFactory tomcatCustomizer() {
    TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
    factory.addConnectorCustomizers(gracefulShutdown());
    return factory;
}
```

到此为止，内置的 Tomcat 容器平滑关闭的操作就完成了，Spring 容器优雅停止上面已经说过了，再次就不再赘述了。

通过测试，同样可以达到上面那样优雅停止的效果。

# 总结

本文主要讲解了 Spring Boot 2.3 版本和旧版本的优雅停止，避免强制停止导致正在处理的业务逻辑会被中断，进而导致产生业务异常的情形。

另外使用 Actuator 的同时要注意安全问题，比如可以通过引入 `security` 依赖，打开安全限制并进行身份验证，设置单独的 Actuator 管理端口并配置只对内网开放等。

本文的完整代码在 `https://github.com/wupeixuan/SpringBoot-Learn` 的 `graceful-shutdown` 目录下。

**最好的关系就是互相成就**，大家的**在看、转发、留言**三连就是我创作的最大动力。

> 参考 
>
> https://github.com/spring-projects/spring-boot/issues/4657
>
> https://github.com/wupeixuan/SpringBoot-Learn