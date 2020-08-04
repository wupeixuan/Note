Spring Boot 集成 Redis 实现数据缓存，只要添加一些注解方法，就可以动态的去操作缓存了，减少代码的操作。

在这个例子中我使用的是 Redis，其实缓存类型还有很多，例如 `Ecache、Mamercache、Caffeine`  等。

## Redis 简介

Redis 是一个开源，高级的键值存储和一个适用的解决方案，用于构建高性能，可扩展的 Web 应用程序。

Redis 相关的知识就不在这里赘述了，感兴趣的可以看下 Redis 系列文章：[Redis 专栏](https://mp.weixin.qq.com/s/jz3nidrFfoH8_XOeoj5b_g)

下面我们在 Spring Boot 中集成 Redis 来实现数据缓存。

## Spring Boot 集成 Redis 实现缓存

Spring Boot 集成 Redis 实现缓存主要分为以下三步：

1. 加入 Redis 依赖
2. 加入 Redis 配置
3. 演示 Redis 缓存

### 加入依赖

首先创建一个项目，在项目中加入 Redis 依赖，项目依赖如下所示（由于使用 Redis 连接池，需额外引入 `commons-pool2`）：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

在 `spring-boot-starter-data-redis` 1.X 版本默认使用 Jedis 客户端，在 2.X 版本默认开始使用 Lettuce 客户端，如果习惯使用 Jedis 的话，可以从 `spring-boot-starter-data-redis` 中排除 Lettuce 并引入 Jedis。

### 加入配置

在配置文件 `application.properties` 中配置 Redis 的相关参数，具体内容如下：

```
#Redis 索引（0~15，默认为 0）
spring.redis.database=0
spring.redis.host=127.0.0.1
#Redis 密码，如果没有就默认不配置此参数
spring.redis.password=
spring.redis.port=6379
#Redis 连接的超时时间
spring.redis.timeout=1000
#连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
#连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.lettuce.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=10
#连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
```

接下来在 `config` 包下创建一个 Redis 配置类 `RedisConfig`，在配置类上加入注解 `@Configuration`，注入一个 `CacheManager` 来配置一些相关信息，代码如下：

```java
@Configuration
public class RedisConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30))
                .prefixKeysWith("cache:user:")
                .disableCachingNullValues()
                .serializeKeysWith(keySerializationPair())
                .serializeValuesWith(valueSerializationPair());
        return RedisCacheManager.builder(factory)
                .withCacheConfiguration("user", redisCacheConfiguration).build();
    }

    private RedisSerializationContext.SerializationPair<String> keySerializationPair() {
        return RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer());
    }

    private RedisSerializationContext.SerializationPair<Object> valueSerializationPair() {
        return RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer());
    }
}
```

首先通过 `RedisCacheConfiguration` 生成默认配置，然后对缓存进行自定义化配置，比如过期时间、缓存前缀、key/value 序列化方法等，然后构建出一个 `RedisCacheManager`，其中通过 `keySerializationPair` 方法为 key 配置序列化，`valueSerializationPair` 为 value 配置序列化。


### 定义用户实体类

在 `domain` 包下创建一个用户实体类：

```java
public class User {

    private Long id;

    private String name;

    private String password;
    // 省略 getter/setter
}
```

### 在服务中使用 SpringCache 注解

在 `service` 包下定义用户接口，分别包含添加用户、查询用户、更新用户以及删除用户四个接口，具体代码如下：

```java
public interface UserService {

    void addUser(User user);

    User getUserById(Long id);

    User updateUser(User user);

    void deleteById(Long id);
}
```

然后编写实现类，为了方便演示，在这里使用 `Map<Long, User> userMap`，没有去连接数据库，其中用到的注解有 `@CacheConfig、@Cacheable、@CachePut 以及 @CacheEvict`，具体代码如下：

```java
@Service
@CacheConfig(cacheNames = "user")
public class UserServiceImpl implements UserService {

    Map<Long, User> userMap = Collections.synchronizedMap(new HashMap<>());

    @Override
    public void addUser(User user) {
        userMap.put(user.getId(), user);
    }

    @Override
    @Cacheable(key = "#id")
    public User getUserById(Long id) {
        if (!userMap.containsKey(id)) {
            return null;
        }
        return userMap.get(id);
    }

    @Override
    @CachePut(key = "#user.id")
    public User updateUser(User user) {
        if (!userMap.containsKey(user.getId())) {
            throw new RuntimeException("不存在该用户");
        }
        User newUser = userMap.get(user.getId());
        newUser.setPassword(user.getPassword());
        userMap.put(newUser.getId(), newUser);
        return newUser;
    }

    @Override
    @CacheEvict(key = "#id")
    public void deleteById(Long id) {
        userMap.remove(id);
    }
}
```

在这里说下这几个注解：

**@CacheConfig 类级别的缓存注解，允许共享缓存名称**

**@Cacheable 触发缓存入口**

一般用于查询操作，根据 key 查询缓存.

1. 如果 key 不存在，查询 db，并将结果更新到缓存中。
2. 如果 key 存在，直接查询缓存中的数据。

**@CacheEvict 触发移除缓存**

根据 key 删除缓存中的数据。

**@CacahePut 更新缓存**

一般用于更新和插入操作，每次都会请求 db，通过 key 去 Redis 中进行操作。 

1. 如果 key 存在，更新内容 
2. 如果 key 不存在，插入内容。

最后，在 `controller` 包下创建一个 `UserController` 类，提供用户 API 接口（未使用数据库），代码如下：

```java
@RestController
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.getUserById(id));
    }

    @PostMapping
    public ResponseEntity<String> createUser(@RequestBody User user) {
        userService.addUser(user);
        return ResponseEntity.status(HttpStatus.CREATED).body("SUCCESS");
    }

    @PutMapping
    public ResponseEntity<User> updateUser(@RequestBody User user) {
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.updateUser(user));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteUser(@PathVariable Long id) {
        userService.deleteById(id);
        return ResponseEntity.status(HttpStatus.CREATED).body("SUCCESS");
    }
}
```

### 启动类添加开启缓存注解

```java
@SpringBootApplication
@EnableCaching
public class RedisCacheApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisCacheApplication.class, args);
    }
}
```

`@EnableCaching` 表明开启缓存，Spring Boot 会自动配置 Redis 缓存的 CacheManager。

启动项目，先调用添加用户接口，添加用户 wupx，id 为 1，在 getUserById 方法打断点，第一次调用 getUser 接口获取用户的时候，会运行到断点处，第二次调用 getUser 不会运行到断点处，而是直接从 Redis 中读取缓存数据。

通过 Redis 可视化工具 RedisDesktopManager 查看，结果如图所示：

![](https://img-blog.csdnimg.cn/20200730003148112.png)

到此为止，我们就完成了 Spring Boot 与 Redis 的集成实现数据缓存。

# 总结

Spring Boot 集成 Redis 实现数据缓存还是比较轻松的，Spring 提供了缓存注解，使用这些注解可以有效简化编程过程，大家可以自己动手实践下。

本文的完整代码在 https://github.com/wupeixuan/SpringBoot-Learn 的 `redis-cache` 目录下。

**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考 
>
> https://github.com/wupeixuan/SpringBoot-Learn