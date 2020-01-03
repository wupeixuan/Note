[TOC]
## Spring IOC AOP

IOC(Inversion of Control)：其思想是反转资源获取的方向. 传统的资源查找方式要求组件向容器发起请求查找资源. 作为回应, 容器适时的返回资源. 而应用了 IOC 之后, 则是容器主动地将资源推送给它所管理的组件, 组件所要做的仅是选择一种合适的方式来接受资源. 这种行为也被称为查找的被动形式

DI(Dependency Injection) — IOC 的另一种表述方式：即组件以一些预先定义好的方式(例如: setter 方法)接受来自如容器的资源注入. 相对于 IOC 而言，这种表述更直接

## Spring容器

在 Spring IOC 容器读取 Bean 配置创建 Bean 实例之前, 必须对它进行实例化. 只有在容器实例化后, 才可以从 IOC 容器里获取 Bean 实例并使用.


Spring 提供了两种类型的 IOC 容器实现. 
- BeanFactory: IOC 容器的基本实现.
- ApplicationContext: 提供了更多的高级特性. 是 BeanFactory 的子接口.
- BeanFactory 是 Spring 框架的基础设施，面向 Spring 本身；ApplicationContext 面向使用 Spring 框架的开发者，几乎所有的应用场合都直接使用 ApplicationContext 而非底层的 BeanFactory
- 无论使用何种方式, 配置文件时相同的.

## 依赖注入

Spring 支持 3 种依赖注入的方式
- 属性注入
- 构造器注入
- 工厂方法注入（很少使用，不推荐）

### 属性注入
属性注入即通过 setter 方法注入Bean 的属性值或依赖的对象

属性注入使用 <property> 元素, 使用 name 属性指定 Bean 的属性名称，value 属性或 <value> 子节点指定属性值 

属性注入是实际应用中最常用的注入方式

```
    <!--属性注入-->
    <bean id="helloWorld" class="com.wupx.bean.HelloWorld">
        <property name="name" value="Spring"></property>
    </bean>
```

### 构造器注入
通过构造方法注入Bean 的属性值或依赖的对象，它保证了 Bean 实例在实例化后就可以使用。

构造器注入在 <constructor-arg> 元素里声明属性, <constructor-arg> 中没有 name 属性

```
    <!--按索引匹配入参-->
    <bean id="car" class="com.wupx.bean.Car">
        <constructor-arg value="奥迪" index="0"></constructor-arg>
        <constructor-arg value="长春一汽" index="1"></constructor-arg>
        <constructor-arg value="5000" index="2"></constructor-arg>
    </bean>

    <!--按类型匹配入参-->
    <bean id="car" class="com.wupx.bean.Car">
        <constructor-arg value="奥迪" type="java.lang.String"></constructor-arg>
        <constructor-arg value="长春一汽" type="java.lang.String"></constructor-arg>
        <constructor-arg value="5000" type="double"></constructor-arg>
    </bean>
```
## 引用其他Bean

```
    <!--引用其他Bean-->
    <bean id="person" class="com.wupx.bean.Person">
        <property name="car" ref="car"></property>
        <property name="name" value="wupx"></property>
        <property name="age" value="18"></property>
    </bean>
```

## Bean 的作用域

类别 | 说明
---|---
Singleton | 在 Spring IOC 容器中仅存在一个 bean 实例 , bean 以单例的方式存在
Prototype | 每调用一次 getBean() 都会生成一个新的实例
Request   | 每次 HTTP 请求都会创建一个新的 bean , 该作用域仅适用于 WebApplicationContext 环境
Session   | 同一个 HTTP Session 共享一个 bean , 不同的 HTTP Session 使用不同的 bean , 该作用域仅适用于 WebApplicationContext 环境


## 组件扫描

特定组件包括:
- @Component: 基本注解, 标识了一个受 Spring 管理的组件
- @Respository: 标识持久层组件
- @Service: 标识服务层(业务层)组件
- @Controller: 标识表现层组件


```
    <!-- 指定 Spring IOC 容器扫描的包-->
    <context:component-scan base-package="com.wupx.annotation">

    </context:component-scan>
```

## 泛型依赖注入

首先创建两个泛型基类：BaseRepository<T> 和 BaseService<T>  

```
public class BaseRepository<T> {
	public void save()
	{
		System.out.println("BaseRepository<T> save...");
	}
}
```

```
public class BaseService<T> {
	@Autowired
	protected BaseRepository<T> repository;
	
	public void add() {
		System.out.println("BaseService<T> save...");
		System.out.println(repository);
		repository.save();
	}
}
```

然后定义一个User类

```
public class User {
 
}
```

UserRepository，继承BaseRepository，使用@Repository进行标注，让IOC容器进行管理，会生成一个userRepository的bean

```
import org.springframework.stereotype.Repository;
 
@Repository
public class UserRepository extends BaseRepository<User> {
 
}
```

然后是UserService类，继承BaseService类，使用@Service进行标注，让IOC容器进行管理，会生成一个userService的bean



```
import org.springframework.stereotype.Service;
 
@Service
public class UserService extends BaseService<User>{
	
}
```

去测试下泛型依赖注入

```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class Main {
 
	public static void main(String[] args) {
		ApplicationContext ctx = new ClassPathXmlApplicationContext("beans-annotation.xml");
		UserService userService = (UserService) ctx.getBean("userService");
		userService.add();
	}
}
```

运行结果
```
BaseService<T> save...

com.wupx.annotation.UserRepository@25bbf683

BaseRepository<T> save...
```

## AspectJ注解实现AOP

AspectJ 支持 5 种类型的通知注解: 
- @Before: 前置通知, 在方法执行之前执行
- @After: 后置通知, 在方法执行之后执行 
- @AfterRunning: 返回通知, 在方法返回结果之后执行
- @AfterThrowing: 异常通知, 在方法抛出异常之后
- @Around: 环绕通知, 围绕着方法执行


在配置文件中添加，启用 AspectJ 注解支持
```
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

添加切面类

```
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

/**
 * @author wupx
 * @date 2019/08/07
 */
@Aspect
@Component
@Order(1)
public class LogAspect {
    @Before("execution(* com.wupx.annotation.*.*.*(..))")
    public void beforeMethod(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getSimpleName();
        String methodName = joinPoint.getSignature().getName();
        System.out.println("The method " + methodName + " of " + className + " begin");
    }
}
```