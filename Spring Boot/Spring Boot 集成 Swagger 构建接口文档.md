在应用开发过程中经常需要对其他应用或者客户端提供 RESTful API 接口，尤其是在版本快速迭代的开发过程中，**修改接口的同时还需要同步修改对应的接口文档**，这使我们总是做着重复的工作，并且如果**忘记修改接口文档**，就可能造成不必要的麻烦。

为了解决这些问题，Swagger 就孕育而生了，那让我们先简单了解下。

## Swagger 简介

Swagger 是一个规范和完整的框架，用于**生成、描述、调用和可视化 RESTful 风格的 Web 服务**。

总体目标是使客户端和文件系统作为服务器，以同样的速度来更新。文件的方法、参数和模型紧密集成到服务器端的代码中，允许 API 始终保持同步。

下面我们在 Spring Boot 中集成 Swagger 来构建强大的接口文档。

## Spring Boot 集成 Swagger

Spring Boot 集成 Swagger 主要分为以下三步：

1. 加入 Swagger 依赖
2. 加入 Swagger 文档配置
3. 使用 Swagger 注解编写 API 文档

### 加入依赖

首先创建一个项目，在项目中加入 Swagger 依赖，项目依赖如下所示：

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```

### 加入配置

接下来在 `config` 包下创建一个 Swagger 配置类 `Swagger2Configuration`，在配置类上加入注解 `@EnableSwagger2`，表明开启 Swagger，注入一个 Docket 类来配置一些 API 相关信息，`apiInfo()` 方法内定义了几个文档信息，代码如下：

```
@Configuration
@EnableSwagger2
public class Swagger2Configuration {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                // swagger 文档扫描的包
                .apis(RequestHandlerSelectors.basePackage("com.wupx.interfacedoc.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("测试接口列表")
                .description("Swagger2 接口文档")
                .version("v1.0.0")
                .contact(new Contact("wupx", "https://www.tianheyu.top", "wupx@qq.com"))
                .license("Apache License, Version 2.0")
                .licenseUrl("http://www.apache.org/licenses/LICENSE-2.0.html")
                .build();
    }
}
```

列举其中几个文档信息说明下：

- title：接口文档的标题
- description：接口文档的详细描述
- termsOfServiceUrl：一般用于存放公司的地址
- version：API 文档的版本号
- contact：维护人、维护人 URL 以及 email
- license：许可证
- licenseUrl：许可证 URL

### 编写 API 文档

在 `domain` 包下创建一个 `User` 实体类，使用 `@ApiModel` 注解表明这是一个 Swagger 返回的实体，`@ApiModelProperty` 注解表明几个实体的属性，代码如下（其中 getter/setter 省略不显示）：

```
@ApiModel(value = "用户", description = "用户实体类")
public class User {

    @ApiModelProperty(value = "用户 id", hidden = true)
    private Long id;

    @ApiModelProperty(value = "用户姓名")
    private String name;

    @ApiModelProperty(value = "用户年龄")
    private String age;

    // getter/setter
}
```

最后，在 `controller` 包下创建一个 `UserController` 类，提供用户 API 接口（未使用数据库），代码如下：

```
@RestController
@RequestMapping("/users")
@Api(tags = "用户管理接口")
public class UserController {

    Map<Long, User> users = Collections.synchronizedMap(new HashMap<>());

    @GetMapping("/")
    @ApiOperation(value = "获取用户列表", notes = "获取用户列表")
    public List<User> getUserList() {
        return new ArrayList<>(users.values());
    }

    @PostMapping("/")
    @ApiOperation(value = "创建用户")
    public String addUser(@RequestBody User user) {
        users.put(user.getId(), user);
        return "success";
    }

    @GetMapping("/{id}")
    @ApiOperation(value = "获取指定 id 的用户")
    @ApiImplicitParam(name = "id", value = "用户 id", paramType = "query", dataTypeClass = Long.class, defaultValue = "999", required = true)
    public User getUserById(@PathVariable Long id) {
        return users.get(id);
    }

    @PutMapping("/{id}")
    @ApiOperation(value = "根据 id 更新用户")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "用户 id", defaultValue = "1"),
            @ApiImplicitParam(name = "name", value = "用户姓名", defaultValue = "wupx"),
            @ApiImplicitParam(name = "age", value = "用户年龄", defaultValue = "18")
    })
    public User updateUserById(@PathVariable Long id, @RequestParam String name, @RequestParam Integer age) {
        User user = users.get(id);
        user.setName(name);
        user.setAge(age);
        return user;
    }

    @DeleteMapping("/{id}")
    @ApiOperation(value = "删除用户", notes = "根据 id 删除用户")
    @ApiImplicitParam(name = "id", value = "用户 id", dataTypeClass = Long.class, required = true)
    public String deleteUserById(@PathVariable Long id) {
        users.remove(id);
        return "success";
    }
}
```

启动项目，访问 `http://localhost:8080/swagger-ui.html`，可以看到我们定义的文档已经在 Swagger 页面上显示了，如下图所示：

![](https://img-blog.csdnimg.cn/20200516161309430.png)

到此为止，我们就完成了 Spring Boot 与 Swagger 的集成。

同时 Swagger 除了**接口文档功能**外，还提供了**接口调试**功能，以创建用户接口为例，单击创建用户接口，可以看到接口定义的**参数、返回值、响应码**等，单击 `Try it out` 按钮，然后点击 `Execute` 就可以发起调用请求、创建用户，如下图所示：

![](https://img-blog.csdnimg.cn/20200516161740984.png)

## 注解介绍

由于 Swagger 2 提供了非常多的注解供开发使用，这里列举一些比较常用的注解。

### @Api

`@Api` 用在接口文档资源类上，用于标记当前类为 Swagger 的文档资源，其中含有几个常用属性：

- value：定义当前接口文档的名称。
- description：用于定义当前接口文档的介绍。
- tag：可以使用多个名称来定义文档，但若同时存在 tag 属性和 value 属性，则 value 属性会失效。
- hidden：如果值为 true，就会隐藏文档。

### @ApiOperation

`@ApiOperation` 用在接口文档的方法上，主要用来注解接口，其中包含几个常用属性：

- value：对API的简短描述。
- note：API的有关细节描述。
- esponse：接口的返回类型（注意：这里不是返回实际响应，而是返回对象的实际结果）。
- hidden：如果值为 true，就会在文档中隐藏。

### @ApiResponse、@ApiResponses

`@ApiResponses` 和 @`ApiResponse` 二者配合使用返回 HTTP 状态码。`@ApiResponses` 的 value 值是 `@ApiResponse` 的集合，多个 `@ApiResponse` 用逗号分隔，其中 `@ApiResponse` 包含的属性如下：

- code：HTTP状态码。
- message：HTTP状态信息。
- responseHeaders：HTTP 响应头。

### @ApiParam

`@ApiParam` 用于方法的参数，其中包含以下几个常用属性：

- name：参数的名称。
- value：参数值。
- required：如果值为 true，就是必传字段。
- defaultValue：参数的默认值。
- type：参数的类型。
- hidden：如果值为 true，就隐藏这个参数。

### @ApiImplicitParam、@ApiImplicitParams

二者配合使用在 API 方法上，`@ApiImplicitParams` 的子集是 `@ApiImplicitParam` 注解，其中 `@ApiImplicitParam` 注解包含以下几个参数：

- name：参数的名称。
- value：参数值。
- required：如果值为 true，就是必传字段。
- defaultValue：参数的默认值。
- dataType：数据的类型。
- hidden：如果值为 true，就隐藏这个参数。
- allowMultiple：是否允许重复。

### @ResponseHeader

API 文档的响应头，如果需要设置响应头，就将 `@ResponseHeader` 设置到 `@ApiResponse` 的 `responseHeaders` 参数中。`@ResponseHeader` 提供了以下几个参数：

- name：响应头名称。
- description：响应头备注。

### @ApiModel

设置 API 响应的实体类，用作 API 返回对象。`@ApiModel` 提供了以下几个参数：

- value：实体类名称。
- description：实体类描述。
- subTypes：子类的类型。

### @ApiModelProperty

设置 API 响应实体的属性，其中包含以下几个参数：

- name：属性名称。
- value：属性值。
- notes：属性的注释。
- dataType：数据的类型。
- required：如果值为 true，就必须传入这个字段。
- hidden：如果值为 true，就隐藏这个字段。
- readOnly：如果值为 true，字段就是只读的。
- allowEmptyValue：如果为 true，就允许为空值。

到此为止，我们就介绍完了 Swagger 提供的主要注解。

# 总结

Swagger 可以轻松地整合到 Spring Boot 中构建出强大的 RESTful API 文档，可以减少我们编写接口文档的工作量，同时接口的说明内容也整合入代码中，可以让我们在修改代码逻辑的同时方便的修改接口文档说明，另外 Swagger 也提供了页面测试功能来调试每个 RESTful API。

如果项目中还未使用，不防尝试一下，会发现效率会提升不少。

本文的完整代码在 https://github.com/wupeixuan/SpringBoot-Learn 的 `interface-doc` 目录下。

**最好的关系就是互相成就**，大家的**在看、转发、留言**三连就是我创作的最大动力。

> 参考 
>
> http://swagger.io
>
> https://github.com/wupeixuan/SpringBoot-Learn
>
> 《Spring Boot 2 实战之旅》