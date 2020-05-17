在项目迭代开发中，难免会有更新数据库 Schema 的情况，比如添加新表、在表中增加字段或者删除字段等，那么当我对数据库进行一系列操作后，如何快速地在其他同事的电脑上同步？如何在测试/生产服务器上快速同步？

![](https://img-blog.csdnimg.cn/20200507113604959.png)

每次发版的时候，由于大家都可能有 sql 更改情况，这样就会有以下痛点：

- 忘记某些 sql 修改
- 每个开发人员的 sql 的执行顺序问题
- 重复更新
- 需要手动去数据库执行脚本

以上问题以及痛点可以通过 Flyway 工具来解决，Flyway 可以实现自动化的数据库版本管理，并且能够记录数据库版本更新记录。

## Flyway 简介

Flyway 是独立于数据库的应用、管理并跟踪数据库变更的数据库版本管理工具。用通俗的话讲，Flyway 可以像 Git 管理不同人的代码那样，管理不同人的 sql 脚本，从而做到数据库同步，更多的信息可以在 Flyway 的官网上进行阅读学习。

另外 Flyway 支持很多关系数据库，具体如下所示：

![](https://img-blog.csdnimg.cn/20200507155532955.png)

下面我们在 Spring Boot 中集成 Flyway 来实现数据库版本控制。

## Spring Boot 集成 Flyway

首先创建一个 SpringBoot 项目，然后在 `pom.xml` 加入如下依赖集成 Flyway：

```
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
    <version>5.2.4</version>
</dependency>
```

然后在 `application.yml` 中写入 mysql 的配置及 Flyway 的相关配置(Flyway locations 默认读取当前项目下的 `resources/db/migration` 目录)

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=123

spring.flyway.locations=classpath:/db/migration/
```

接下来，在 `resources/db/migration` 目录下创建需要执行的 SQL 脚本即可。

其中，SQL 脚本命名规范如下：

![](https://img-blog.csdnimg.cn/20200507112235596.png)

- Prefix 前缀：V 代表版本迁移，U 代表撤销迁移，R 代表可重复迁移
- Version 版本号：版本号通常 `.` 和整数组成
- Separator 分隔符：固定由两个下划线 `__` 组成
- Description 描述：由下划线分隔的单词组成，用于描述本次迁移的目的
- Suffix 后缀：如果是 SQL 文件那么固定由 `.sql` 组成，如果是基于 Java 类则默认不需要后缀

那么，我们按照命名规范在 `resources/db/migration` 目录下，创建 `V1.0__init_db.sql` SQL 迁移脚本，具体内容如下：

```
DROP TABLE IF EXISTS `user` ;

CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(20) NOT NULL COMMENT '姓名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `user` (`id`, `name`, `age`) VALUES ('1', 'wupx', '18');
```

最后启动项目，执行日志如下所示：

```
2020-05-07 12:41:29.126  INFO 13732 --- [           main] o.f.c.internal.license.VersionPrinter    : Flyway Community Edition 5.2.4 by Boxfuse
2020-05-07 12:41:29.236  INFO 13732 --- [           main] o.f.c.internal.database.DatabaseFactory  : Database: jdbc:mysql://localhost:3306/test (MySQL 5.5)
2020-05-07 12:41:29.287  INFO 13732 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 1 migration (execution time 00:00.009s)
2020-05-07 12:41:29.330  INFO 13732 --- [           main] o.f.c.i.s.JdbcTableSchemaHistory         : Creating Schema History table: `test`.`flyway_schema_history`
2020-05-07 12:41:29.479  INFO 13732 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema `test`: << Empty Schema >>
2020-05-07 12:41:29.480  INFO 13732 --- [           main] o.f.core.internal.command.DbMigrate      : Migrating schema `test` to version 1.0 - init db
2020-05-07 12:41:29.481  WARN 13732 --- [           main] o.f.c.i.s.DefaultSqlScriptExecutor       : DB: Unknown table 'user' (SQL State: 42S02 - Error Code: 1051)
2020-05-07 12:41:29.631  INFO 13732 --- [           main] o.f.core.internal.command.DbMigrate      : Successfully applied 1 migration to schema `test` (execution time 00:00.301s)
```

从启动日志中可以看出，Flyway 监测到需要运行版本脚本来初始化数据库，因此执行了 `V1.0__init_db.sql` 脚本，从而创建了 `user` 表，另外还自动创建了 `flyway_schema_history` 表，用于记录所有版本演化和状态，其表结构如下(以 MySQL 为例)：

| Field          | Type          | Null | Key | Default           |
|----------------|---------------|------|-----|-------------------|
| version_rank   | int(11)       | NO   | MUL | NULL              |
| installed_rank | int(11)       | NO   | MUL | NULL              |
| version        | varchar(50)   | NO   | PRI | NULL              |
| description    | varchar(200)  | NO   |     | NULL              |
| type           | varchar(20)   | NO   |     | NULL              |
| script         | varchar(1000) | NO   |     | NULL              |
| checksum       | int(11)       | YES  |     | NULL              |
| installed_by   | varchar(100)  | NO   |     | NULL              |
| installed_on   | timestamp     | NO   |     | CURRENT_TIMESTAMP |
| execution_time | int(11)       | NO   |     | NULL              |
| success        | tinyint(1)    | NO   | MUL | NULL              |

查询 `flyway_schema_history` 表，发现增加了一条版本号为 `1.0` 的，使用 `V1.0__init_db.sql` 迁移脚本的记录。

```
mysql> SELECT * FROM flyway_schema_history;
+----------------+---------+-------------+------+-------------------+------------+--------------+---------------------+----------------+---------+
| installed_rank | version | description | type | script            | checksum   | installed_by | installed_on        | execution_time | success |
+----------------+---------+-------------+------+-------------------+------------+--------------+---------------------+----------------+---------+
|              1 | 1.0     | init db     | SQL  | V1.0__init_db.sql | 1317299633 | root         | 2020-05-07 12:41:29 |             97 |       1 |
+----------------+---------+-------------+------+-------------------+------------+--------------+---------------------+----------------+---------+
```

再去查询 `user` 表，发现 sql 脚本中的数据也插入成功了。

```
mysql> SELECT * FROM user;
+----+------+-----+
| id | name | age |
+----+------+-----+
|  1 | wupx |  18 |
+----+------+-----+
```


接下来再次运行项目，结果如下：

```
2020-05-07 15:34:49.843  INFO 41880 --- [           main] o.f.c.internal.license.VersionPrinter    : Flyway Community Edition 5.2.4 by Boxfuse
2020-05-07 15:34:49.981  INFO 41880 --- [           main] o.f.c.internal.database.DatabaseFactory  : Database: jdbc:mysql://localhost:3306/test (MySQL 5.5)
2020-05-07 15:34:50.036  INFO 41880 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 1 migration (execution time 00:00.013s)
2020-05-07 15:34:50.043  INFO 41880 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema `test`: 1.0
2020-05-07 15:34:50.043  INFO 41880 --- [           main] o.f.core.internal.command.DbMigrate      : Schema `test` is up to date. No migration necessary.
```

从日志中可以看出，Flyway 发现一个迁移脚本，也就是 `V1.0__init_db.sql`，经过判断已经到达最新版本 1.0，无需执行迁移。

接下来，我们在 `V1.0__init_db.sql` 迁移脚本中添加一条 INSERT 操作：`INSERT INTO `user` (`id`, `name`, `age`) VALUES ('2', 'huxy', '18');` ，再次启动项目，会报如下错误：

```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'flywayInitializer' defined in class path resource [org/springframework/boot/autoconfigure/flyway/FlywayAutoConfiguration$FlywayConfiguration.class]: Invocation of init method failed; nested exception is org.flywaydb.core.api.FlywayException: Validate failed: Migration checksum mismatch for migration version 1.0
-> Applied to database : 1317299633
-> Resolved locally    : -1582367361
```

这个错误的原因就是 Flyway 会给脚本计算一个 checksum 保存在数据库中，用于在之后运行过程中对比 sql 文件是否有变化，如果发生了变化，则会报错，也就防止了误修改脚本导致发生问题。

# 总结

Flyway 可以有效改善数据库版本管理方式，并且是一款 Java 开源的数据库迁移管理工具，具有轻便小巧的特点，可以无门槛快速集成到项目中，如果项目中还未使用，不防尝试一下，想了解更多的可以去官网查看文档学习。

本文的完整代码在 https://github.com/wupeixuan/SpringBoot-Learn 的 `database-version-control` 目录下。

**最好的关系就是互相成就**，大家的**在看、转发、留言**三连就是我创作的最大动力。

> 参考 
>
> https://flywaydb.org/
>
> https://github.com/wupeixuan/SpringBoot-Learn