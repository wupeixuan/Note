项目开发中经常需要执行一些定时任务，比如在每天凌晨，需要从 implala 数据库拉取产品功能活跃数据，分析处理后存入到 MySQL 数据库中。类似这样的需求还有许多，那么怎么去实现定时任务呢，有以下几种实现方式。

# Java 定时任务的几种实现方式
- 基于 java.util.Timer 定时器，实现类似闹钟的定时任务
- 使用 Quartz、elastic-job、xxl-job 等开源第三方定时任务框架，适合分布式项目应用
- 使用 Spring 提供的一个注解： @Schedule，开发简单，使用比较方便，也是本文介绍的一种方式

Spring 自身提供了对定时任务的支持，本文将介绍 Spring Boot 中 @Scheduled 定时器的使用。

# 创建定时任务
首先，在项目启动类上添加 @EnableScheduling 注解，开启对定时任务的支持
 
```
@SpringBootApplication
@EnableScheduling
public class ScheduledApplication {

	public static void main(String[] args) {
		SpringApplication.run(ScheduledApplication.class, args);
	}

}
```

其中  @EnableScheduling注解的作用是发现注解@Scheduled的任务并后台执行。

其次，编写定时任务类和方法，定时任务类通过 Spring IOC 加载，使用 @Component 注解，定时方法使用 @Scheduled 注解。

```
@Component
public class ScheduledTask {

    @Scheduled(fixedRate = 3000)
    public void scheduledTask() {
        System.out.println("任务执行时间：" + LocalDateTime.now());
    }

}
```

fixedRate 是 long 类型，表示任务执行的间隔毫秒数，以上代码中的定时任务每 3 秒执行一次。

运行定时工程，项目启动和运行日志如下，可见每 3 秒打印一次日志执行记录。

```
2019-10-16 22:50:04.791  INFO 10610 --- [           main] com.wupx.ScheduledApplication                : Started ScheduledApplication in 1.513 seconds (JVM running for 1.976)
任务执行时间：2019-10-16T22:50:04.791
任务执行时间：2019-10-16T22:50:07.782
任务执行时间：2019-10-16T22:50:10.779
```

# @Scheduled详解
在上面的入门例子中，使用了@Scheduled(fixedRate = 3000) 注解来定义每过 3 秒执行的任务，对于 @Scheduled 的使用可以总结如下几种方式：

- @Scheduled(fixedRate = 3000) ：上一次开始执行时间点之后 3 秒再执行（fixedRate 属性：定时任务开始后再次执行定时任务的延时（需等待上次定时任务完成），单位毫秒）
- @Scheduled(fixedDelay = 3000) ：上一次执行完毕时间点之后 3 秒再执行（fixedDelay 属性：定时任务执行完成后再次执行定时任务的延时（需等待上次定时任务完成），单位毫秒）
- @Scheduled(initialDelay = 1000, fixedRate = 3000) ：第一次延迟1秒后执行，之后按fixedRate的规则每 3 秒执行一次（initialDelay 属性：第一次执行定时任务的延迟时间，需配合fixedDelay或者fixedRate来使用）
- @Scheduled(cron="0 0 2 1 * ? *") ：通过cron表达式定义规则

其中，常用的cron表达式有：
- 0 0 2 1 * ? * ：表示在每月 1 日的凌晨 2 点执行
- 0 15 10 ? * MON-FRI ：表示周一到周五每天上午 10:15 执行
- 0 15 10 ? 6L 2019-2020 ：表示 2019-2020 年的每个月的最后一个星期五上午 10:15 执行
- 0 0 10,14,16 * * ? ：每天上午 10 点，下午 2 点，4 点执行
- 0 0/30 9-17 * * ? ：朝九晚五工作时间内每半小时执行
- 0 0 12 ? * WED ：表示每个星期三中午 12 点执行
- 0 0 12 * * ? ：每天中午 12点执行
- 0 15 10 ? * * ：每天上午 10:15 执行
- 0 15 10 * * ? ：每天上午 10:15 执行
- 0 15 10 * * ? * ：每天上午 10:15 执行
- 0 15 10 * * ? 2019 ：2019 年的每天上午 10:15 执行


# 总结
本文主要介绍了基于 Spring Boot 内置的定时任务的配置使用，主要涉及两个注解，四个属性的配置：
- 主程序入口 @EnableScheduling 开启定时任务
- 定时方法上 @Scheduled 设置定时
- cron属性：按cron规则执行
- fixedRate 属性：以固定速率执行
- fixedDelay 属性：上次执行完毕后延迟再执行
- initialDelay 属性：第一次延时执行，第一次执行完毕后延迟后再次执行

<center><img src="https://img2018.cnblogs.com/blog/1356806/201910/1356806-20191009000648748-355850292.png" /></center>

> 参考
> 
> https://spring.io/guides/gs/scheduling-tasks/
> 
> https://www.tutorialspoint.com/spring_boot/spring_boot_scheduling.htm
> 
> https://docs.spring.io/spring/docs/2.5.x/reference/scheduling.html