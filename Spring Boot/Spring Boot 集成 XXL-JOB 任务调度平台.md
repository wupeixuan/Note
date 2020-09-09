在开发中需要将已有的定时任务抽离出来，方便管理查看，因此选择集成分布式任务调度平台 XXL-JOB，本文就讲解下 Spring Boot 如何集成 XXL-JOB 任务调度平台。

## XXL-JOB 简介

XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

下面我们在 Spring Boot 中集成 XXL-JOB 来完成定时任务的编写（本文选择的 XXL-JOB 版本为 2.2.0）。

## Spring Boot 集成 XXL-JOB

Spring Boot 集成 XXL-JOB 主要分为以下两步：

1. 配置运行调度中心（xxl-job-admin）
2. 配置运行执行器项目（xxl-job-executor）

### 配置运行调度中心

首先从源码仓库中下载代码，代码地址有两个：

1. GitHub：https://github.com/xuxueli/xxl-job
2. Gitee：http://gitee.com/xuxueli0323/xxl-job

下载完之后，在 `doc/db` 目录下有数据库脚本 `tables_xxl_job.sql`，执行下脚本初始化调度数据库 `xxl_job`，如下图所示：

![](https://img-blog.csdnimg.cn/20200909145602247.png)

可以根据需要修改 xxl-job-admin 的配置文件，主要是修改数据源信息，若需要用到邮件报警功能，需要配置邮箱。

然后启动项目，正常启动后，访问地址为：`http://localhost:8080/xxl-job-admin`，默认的账户为 admin，密码为 123456，访问后台管理系统后台，界面如下：

![](https://img-blog.csdnimg.cn/20200906175436997.png)

这样就表示调度中心已经搞定了，下一步就是创建执行器项目。

### 配置运行执行器项目

创建一个项目，在项目中加入 xxl-job-core 依赖，项目依赖如下所示：

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>com.xuxueli</groupId>
	<artifactId>xxl-job-core</artifactId>
	<version>2.2.0</version>
</dependency>
```

#### 加入配置

在配置文件 `application.properties` 中配置 xxl-job 执行器的相关参数，具体内容如下：

```
### 调度中心部署跟地址 [选填]：如调度中心集群部署存在多个地址则用逗号分隔。执行器将会使用该地址进行"执行器心跳注册"和"任务结果回调"；为空则关闭自动注册；
xxl.job.admin.addresses=http://127.0.0.1:8080/xxl-job-admin
### 执行器通讯TOKEN [选填]：非空时启用；
xxl.job.accessToken=
### 执行器AppName [选填]：执行器心跳注册分组依据；为空则关闭自动注册
xxl.job.executor.appname=xxl-job-executor
### 执行器注册 [选填]：优先使用该配置作为注册地址，为空时使用内嵌服务 ”IP:PORT“ 作为注册地址。从而更灵活的支持容器类型执行器动态IP和动态映射端口问题。
xxl.job.executor.address=
### 执行器IP [选填]：默认为空表示自动获取IP，多网卡时可手动设置指定IP，该IP不会绑定Host仅作为通讯时用；地址信息用于 "执行器注册" 和 "调度中心请求并触发任务"；
xxl.job.executor.ip=
### 执行器端口号 [选填]：小于等于0则自动获取；默认端口为9999，单机部署多个执行器时，注意要配置不同执行器端口；
xxl.job.executor.port=9999
### 执行器运行日志文件存储磁盘路径 [选填] ：需要对该路径拥有读写权限；为空则使用默认路径；
xxl.job.executor.logpath=/data/applogs/xxl-job/jobhandler
### 执行器日志文件保存天数 [选填] ： 过期日志自动清理, 限制值大于等于3时生效; 否则, 如-1, 关闭自动清理功能；
xxl.job.executor.logretentiondays=30
```

其中指定了上一步部署的调度中心的地址和执行器的相关参数。

然后在 config 包下创建 `XxlJobConfiguration` 类，会从配置文件中读取到对应的参数，接着申明一个 `xxlJobExecutor` 方法，返回的是一个 `XxlJobSpringExecutor`，这个方法主要是如何初始化并创建一个 `XxlJobSpringExecutor`。

```
@Configuration
public class XxlJobConfiguration {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfiguration.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.accessToken}")
    private String accessToken;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.address}")
    private String address;

    @Value("${xxl.job.executor.ip}")
    private String ip;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Value("${xxl.job.executor.logpath}")
    private String logPath;

    @Value("${xxl.job.executor.logretentiondays}")
    private int logRetentionDays;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        // registry jobhandler
        XxlJobSpringExecutor.registJobHandler("beanClassJobHandler", new BeanClassJobHandler());
        // init executor
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setAddress(address);
        xxlJobSpringExecutor.setIp(ip);
        xxlJobSpringExecutor.setPort(port);
        xxlJobSpringExecutor.setAccessToken(accessToken);
        xxlJobSpringExecutor.setLogPath(logPath);
        xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);
        return xxlJobSpringExecutor;
    }
}
```

接下来就可以创建任务了。

#### 编写 JobHandler

在这里主要演示下 Bean 模式的任务，可以基于类和方法进行开发，下面先介绍基于类的开发的任务。

##### BEAN模式（类形式）

首先创建一个类 `BeanClassJobHandler`，继承 `IJobHandler` 实现 `execute` 方法，然后通过 `XxlJobLogger.log` 来打印日志。

```
@Component
public class BeanClassJobHandler extends IJobHandler {

    @Override
    public ReturnT<String> execute(String param) throws Exception {
        XxlJobLogger.log("bean class jobhandler running...");
        return ReturnT.SUCCESS;
    }
}
```

基于类开发的任务需要手动注册到执行器工厂，具体代码如下所示：

```
XxlJobSpringExecutor.registJobHandler("beanClassJobHandler", new BeanClassJobHandler());
```

到此一个任务就开发完成了，下面介绍下基于方法的开发方式：

##### BEAN模式（方法形式）

基于方法开发的任务比较简单，编写一个方法，并添加 `@XxlJob` 注解，会自动扫描该任务并注入到执行器容器。

```
@Component
public class BeanMethodJobHandler {

    @XxlJob("beanMethodJobHandler")
    public ReturnT<String> beanMethodJobHandler(String param) throws Exception {
        XxlJobLogger.log("bean method jobhandler running...");
        return ReturnT.SUCCESS;
    }
}
```

至此，执行器项目就开发完成了，启动项目，在执行器管理页面添加该执行器。

![新增执行器](https://img-blog.csdnimg.cn/20200909213810890.png)

执行器添加完成后，需要在任务管理界面添加我们刚才开发的两个任务，下面以 BEAN 模式方法任务为例：

![新增任务](https://img-blog.csdnimg.cn/20200909215148238.png)

点击保存后，一个定时任务就完成了，是不是很简单呢？

下面启动任务来查看下执行结果，在这里点击“执行一次”，然后查询执行日志，结果如下图：

![执行日志](https://img-blog.csdnimg.cn/20200909215709124.png)

可以看到我们的任务已经成功执行了，至此，Spring Boot 集成 XXL-JOB 任务调度平台就完成了。

# 总结

Spring Boot 与 XXL-JOB 的集成是不是很简单呢？在这里只是简单地入门，想要了解更多可以看下官方文档：`https://www.xuxueli.com/xxl-job`。

还没有使用过的可以通过本文快速上手，来实操起来吧！

本文的完整代码在 `https://github.com/wupeixuan/SpringBoot-Learn` 的 `xxl-job-executor` 目录下。

**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考 
>
> https://www.xuxueli.com/xxl-job
>
> https://github.com/wupeixuan/SpringBoot-Learn