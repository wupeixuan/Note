在阅读《阿里巴巴Java开发手册》时，发现有一条关于前后端超大整数返回的规约，具体内容如下：

![](https://img-blog.csdnimg.cn/20200911110331712.png)

这个问题在之前和前端联调的时候发生过，发现根据脚本 id 去审批的时候，状态没有变化，后来和前端沟通后，才知道这是 JavaScript 的一个坑，下面来复现下这个错误：

# 错误演示

创建一个 Spring Boot 项目，然后在新建一个接口，可以返回 DbScript 对象，其中 `id` 是由 mybatis-plus 的 `IdWorker.getId`（基于 Snowflake 算法）生成的 19 位 `long` 类型的数值。

```
@RestController
@RequestMapping("/dbScrip")
public class DbScriptController {
    Logger logger = LoggerFactory.getLogger(DbScriptController.class);

    @RequestMapping("/info")
    public DbScript getDbScript() {
        DbScript dbScript = new DbScript();
        // 赋予一个大整数 long 型脚本 id
        long id = IdWorker.getId();
        dbScript.setId(id);
        logger.info("id:{}", id);
        return dbScript;
    }
}
```

接着启动服务，在浏览器上访问该接口，结果如下所示：

![](https://img-blog.csdnimg.cn/20200911121551478.png)

通过日志可以看到后端传给前端的 `id` 为 `1304270071757017088`，但是前端拿到的却为 `1304270071757017000`，其中发生了精度损失。

为什么会发生这样的情况呢？

通过开发手册，我们可以知道如果返回的数值超过 2 的 53 次方，就会转换成 JS 的 Number，此时有些数值就有可能发生精度损失。

# 解决方法

那如果遇到了这种情况，该如何解决呢？

不要慌，可以采取以下几种方法：

1. 如果这个对象只在这个方法中用到了，可以将该属性直接从 `Long` 类型改为 `String` 类型。
2. 如果这个对象在很多地方都用到了，可以在序列化的时候，将 `Long` 类型转换成 `String` 类型。
3. 还可以添加一个新的 `String` 类型的属性，专门用来在前后端传输这种大整数。

## 第一种方法

第一种方法比较简单，直接将 `Long id;` 改为 `String id;`，这种只适用于这个对象只在这个方法中使用了，比较局限。

## 第二种方法

第二种方法可以在属性上增加注解，如果使用的 `jackson`，可以添加 `@JsonFormat(shape = JsonFormat.Shape.STRING)` 或者 `@JsonSerialize(using = ToStringSerializer.class)` 注解。

如果这种需要修改的情况比较多，那么逐个添加还是有点费事，那么还有什么好办法吗？

如果使用的是  `jackson`，它有个配置参数 `WRITE_NUMBERS_AS_STRINGS`，可以强制将所有数字全部转成字符串输出，使用方法很简单，只需要配置参数即可：`spring.jackson.generator.write_numbers_as_strings=true`，这种方式的优点是使用方便，不需要调整代码；缺点是颗粒度太大，所有的数字都被转成字符串输出了，包括按照 `timestamp` 格式输出的时间也是如此。

那么还有什么方法能够只对 `Long` 类型进行处理转换成 String 类型呢？

Jackson 提供了这种支持，可以对 `ObjectMapper` 进行定制，具体代码如下所示：

```
public class JacksonConfiguration {

    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jackson2ObjectMapperBuilderCustomizer() {
        return jacksonObjectMapperBuilder -> jacksonObjectMapperBuilder
                .serializerByType(Long.class, ToStringSerializer.instance)
                .serializerByType(Long.TYPE, ToStringSerializer.instance);
    }
}
```

通过定义 `Jackson2ObjectMapperBuilderCustomizer`，对 `Jackson2ObjectMapperBuilder` 对象进行定制，对 `Long` 型数据进行了定制，使用`ToStringSerializer`来进行序列化。

## 第三种方法

第三种方法就需要多一个属性，比如使用`String dbScripId`，用来代替之前的 `id`。

# 总结

本文针对《阿里巴巴Java开发手册》中的对于需要使用超大整数的场景，服务端一律使用 String 字符串类型返回，禁止使用
`Long` 类型出发，提出了几种解决方法，大家可以根据自己的需求去选择方法，有其他解决方法的也欢迎留言讨论。

**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考
>
> 《阿里巴巴Java开发手册（嵩山版）》
>