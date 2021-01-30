策略模式作为一种软件设计模式，指对象有某个行为，但是在不同的场景中，该行为有不同的实现算法，可以替代代码中大量的 if-else。

比如我们生活中的场景：买东西结账可以使用微信支付、支付宝支付或者银行卡支付，这些交易方式就是不同的策略。

那么在什么时候使用策略模式呢？

在《阿里巴巴Java开发手册》中有提到当超过 3 层的 if-else 的逻辑判断代码可以使用策略模式来实现。

![](https://wupx-1256189981.file.myqcloud.com/img/202101/30/1611992446.png)

在 Spring 中实现策略模式的方式有很多种，下面通过一个案例来演示下，比如有个需求需要实现支持第三方登录，目前需要支持以下三种登录方式：

- 微信登录
- QQ 登录
- 微博登录

下面将通过策略模式来实现这个需求，其中策略模式结构如下图所示：

策略模式结构如下图所示：

![策略模式结构](https://wupx-1256189981.file.myqcloud.com/img/202101/30/1611997590.png)

主要包括一个登录接口类和几种登录方式的实现方式，并利用简单工厂来获取对应的处理器。

## 定义策略接口

首先定义一个登录的策略接口 `LoginHandler`，其中包括两个方法：

1. 获取策略类型的方法
2. 处理策略逻辑的方法

```java
public interface LoginHandler<T extends Serializable> {

    /**
     * 获取登录类型
     *
     * @return
     */
    LoginType getLoginType();

    /**
     * 登录
     *
     * @param request
     * @return
     */
    LoginResponse<String, T> handleLogin(LoginRequest request);
}
```

其中，`LoginHandler` 的 `getLoginType ` 方法用来获取登录的类型（即策略类型），用于根据客户端传递的参数直接获取到对应的策略实现。

客户端传递的相关参数都被封装为 LoginRequest，传递给 handleLogin 进行处理。

```java
@Data
public class LoginRequest {

    private LoginType loginType;

    private Long userId;
}
```

其中，根据需求定义登录类型枚举如下：

```java
public enum LoginType {
    QQ,
    WE_CHAT,
    WEI_BO;
}
```

## 实现策略接口

在定义好策略接口后，我们就需要根据各种第三方登录来实现对应的处理逻辑就可以了。

### 微信登录

```java
@Component
public class WeChatLoginHandler implements LoginHandler<String> {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 获取登录类型
     *
     * @return
     */
    @Override
    public LoginType getLoginType() {
        return LoginType.WE_CHAT;
    }

    /**
     * 登录
     *
     * @param request
     * @return
     */
    @Override
    public LoginResponse<String, String> handleLogin(LoginRequest request) {
        logger.info("微信登录：userId：{}", request.getUserId());
        String weChatName = getWeChatName(request);
        return LoginResponse.success("微信登录成功", weChatName);
    }

    private String getWeChatName(LoginRequest request) {
        return "wupx";
    }
}
```

### QQ 登录

```java
@Component
public class QQLoginHandler implements LoginHandler<Serializable> {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 获取登录类型
     *
     * @return
     */
    @Override
    public LoginType getLoginType() {
        return LoginType.QQ;
    }

    /**
     * 登录
     *
     * @param request
     * @return
     */
    @Override
    public LoginResponse<String, Serializable> handleLogin(LoginRequest request) {
        logger.info("QQ登录：userId：{}", request.getUserId());
        return LoginResponse.success("QQ登录成功", null);
    }
}
```

### 微博登录

```java
@Component
public class WeiBoLoginHandler implements LoginHandler<Serializable> {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 获取登录类型
     *
     * @return
     */
    @Override
    public LoginType getLoginType() {
        return LoginType.WEI_BO;
    }

    /**
     * 登录
     *
     * @param request
     * @return
     */
    @Override
    public LoginResponse<String, Serializable> handleLogin(LoginRequest request) {
        logger.info("微博登录：userId：{}", request.getUserId());
        return LoginResponse.success("微博登录成功", null);
    }
}
```

## 创建策略的简单工厂

```java
@Component
public class LoginHandlerFactory implements InitializingBean, ApplicationContextAware {
    private static final Map<LoginType, LoginHandler<Serializable>> LOGIN_HANDLER_MAP = new EnumMap<>(LoginType.class);
    private ApplicationContext appContext;

    /**
     * 根据登录类型获取对应的处理器
     *
     * @param loginType 登录类型
     * @return 登录类型对应的处理器
     */
    public LoginHandler<Serializable> getHandler(LoginType loginType) {
        return LOGIN_HANDLER_MAP.get(loginType);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        // 将 Spring 容器中所有的 LoginHandler 注册到 LOGIN_HANDLER_MAP
        appContext.getBeansOfType(LoginHandler.class)
                .values()
                .forEach(handler -> LOGIN_HANDLER_MAP.put(handler.getLoginType(), handler));
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        appContext = applicationContext;
    }
}
```

我们让 `LoginHandlerFactory`实现 `InitializingBean` 接口，在 `afterPropertiesSet` 方法中，基于 Spring 容器将所有 `LoginHandler` 自动注册到 `LOGIN_HANDLER_MAP`，从而 Spring 容器启动完成后， getHandler 方法可以直接通过 `loginType` 来获取对应的登录处理器。

## 创建登录服务

在登录服务中，我们通过 `LoginHandlerFactory` 来获取对应的登录处理器，从而处理不同类型的第三方登录：

```java
@Service
public class LoginServiceImpl implements LoginService {
    @Autowired
    private LoginHandlerFactory loginHandlerFactory;

    @Override
    public LoginResponse<String, Serializable> login(LoginRequest request) {
        LoginType loginType = request.getLoginType();
        // 根据 loginType 找到对应的登录处理器
        LoginHandler<Serializable> loginHandler =
                loginHandlerFactory.getHandler(loginType);
        // 处理登录
        return loginHandler.handleLogin(request);
    }
}
```

Factory 只负责获取 Handler，Handler 只负责处理具体的登录，Service 只负责逻辑编排，从而达到功能上的低耦合高内聚。

## 测试

写一个 Controller：

```java
@RestController
public class LoginController {

    @Autowired
    private LoginService loginService;

    /**
     * 登录
     */
    @PostMapping("/login")
    public LoginResponse<String, Serializable> login(@RequestParam LoginType loginType, @RequestParam Long userId) {
        LoginRequest loginRequest = new LoginRequest();
        loginRequest.setLoginType(loginType);
        loginRequest.setUserId(userId);
        return loginService.login(loginRequest);
    }
}
```

然后用 Postman 测下下：

![微信登录](https://wupx-1256189981.file.myqcloud.com/img/202101/30/1611995605.png)

![QQ登录](https://wupx-1256189981.file.myqcloud.com/img/202101/30/1611995646.png)

是不是很简单呢？如果需求又要加需求，需要支持 GitHub 第三方登录。

此时我们只需要添加一个新的策略实现，然后在登录枚举中加入对应的类型即可：

```
@Component
public class GitHubLoginHandler implements LoginHandler<Serializable> {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * 获取登录类型
     *
     * @return
     */
    @Override
    public LoginType getLoginType() {
        return LoginType.GIT_HUB;
    }

    /**
     * 登录
     *
     * @param request
     * @return
     */
    @Override
    public LoginResponse<String, Serializable> handleLogin(LoginRequest request) {
        logger.info("GitHub登录：userId：{}", request.getUserId());
        return LoginResponse.success("GitHub登录成功", null);
    }
}
```

此时不需要修改任何代码 ，因为 Spring 容器重启时会自动将 `GitHubLoginHandler` 注册到 `LoginHandlerFactory` 中，使用 Spring 实现策略模式就是这么简单，还不快学起来！