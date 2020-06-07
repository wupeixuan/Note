听说隔壁用 Lombok 的六点就下班了，我也想六点下班！

好的，那么这篇文章就介绍下**什么是 Lombok**，**Lombok 做了什么**以及 **Lombok 是怎么做的**？

在介绍之前，先通过是否使用 Lombok 的效果来看下对比，首先来看下没有 Lombok 之前，我们的一个简单的 Java 对象（POJO）是长什么样子的：

![](https://img-blog.csdnimg.cn/20200329190359294.png)

哦，我的天啊，居然 60 行，好长啊！那我们接下来使用的 Lombok 来试下：

![](https://img-blog.csdnimg.cn/20200329191325512.png)

什么，只使用了 `@Data` 注解就可以实现之前 60 行的相同功能，代码长度整整缩小了 3 倍，这么神奇的嘛？那么让我们走进 Lombok 吧！

## 什么是 Lombok？

下面是 Lombok 官网的简介：

![Lombok 简介](https://img-blog.csdnimg.cn/20200329202549126.png)

简而言之就是 Lombok 是一个很方便的插件，本质是个 Java 库，使用它通过相关注解就可以不用再编写冗长的 getter 或者 equals 等方法了。

接下来讲下 Lombok 实现的原理，这样就知道为什么要这样使用 Lombok 的注解了。

## Lombok 实现原理

要讲 Lombok 的实现原理，在此之前就需要来说下注解的两种解析方式：**运行时注解**和**编译时注解**。

首先来看下**运行时解析**，比如 Spring 配置的 AOP 切面这些注解都是在程序运行的时候通过反射来获取的注解值，但是只有在程序运行时才能获取到这些注解值，导致运行时代码效率很低，并且如果想在编译阶段利用这些注解来进行检查，比如对用户的不合理代码作出错误报告，反射的方法就行不通了。

这就引出了第二种在**编译时解析**，Lombok 工具就是运行在编译时解析的。

那如何把注解与 Java 编译器结合使用呢？Java 也提供了两种解决方案：

第一种方案是**注解处理器（Annotation Processing Tool）**，它最早是在 JDK 1.5 与**注解（Annotation）** 一起引入的，它是一个命令行工具，能够提供构建时基于源代码对程序结构的读取功能，能够通过运行注解处理器来生成新的中间文件，进而影响编译过程，不过它在 JDK 1.8 中被移除了，取而代之的是 **JSR 269 插入式注解处理器（Pluggable Annotation Processing API）**，它是实现了 **JSR 269** 的机制，作为注解处理器的替代方案。

我们通过一个流程图来进一步说明注解处理器的工作原理：

![](https://img-blog.csdnimg.cn/20200329211030587.png)

首先写完代码后会调用 `javac` 编译，在编译后会**生成抽象语法树（AST）**，之后会调用**插入式注解处理器处理**，上面说了插入式注解处理器会修改语法树，生成一些额外的代码，经过处理器的处理语法树会有变动，有变动之后，会再次到生成抽象语法树的处理环节，将变动后的代码再次生成抽象语法树，接着再通过注解处理器，如果这次语法树没有被修改，那么就会**生成响应的字节码**，变成 **class 文件**，以上就是整个注解处理器在整个 `javac` 编译源代码生成 class 文件中起到的作用。

在简单了解了 Lombok 实现原理后，让我们看下 Lombok 有哪些常见的注解：

## Lombok 注解

下面是整理的常用的 Lombok 注解思维导图：

![Lombok 注解](https://img-blog.csdnimg.cn/20200329213331750.png)

右侧上方的 `@Getter、@Setter、@ToString、@EqualsAndHashCode` 这几个名字大家都不陌生，无非就是帮我们生成对应的方法，这四个注解的总和也就是刚开始用的注解 `@Data`，这些注解都归结为常见方法的注解。

右侧下方的 `@AllArgsConstructor、@RequiredArgsConstructor、@NoArgsConstructor` 分别为全参构造函数、必须参数构造函数、无参构造函数，它们通常为构造方法的注解。

左侧的 `@NonNull` 会自动生成空值校验；`@CleanUp` 会自动调用变量的 `close` 方法释放资源；`@Builder` 会自动生成构造者模式，方便对属性 `set/get` 操作； `@Synchronized` 会自动生成同步锁；`@SneakyThrows` 会自动生成 `try/catch` 捕捉异常；`@Slf4j` 是日志相关的，会自动为类添加日志支持。

以上就是 Lombok 为我们提供的比较常用的注解。

## Lombok 使用

首先需要安装 Lombok 插件，我在这里是以 IDEA 2019.3.1 版本来演示的：

### 安装 Lombok 插件

点击 File->Settings->Plugins，搜索 `Lombok`，然后点击安装 Lombok 插件：

![](https://img-blog.csdnimg.cn/20200329214733735.png)

在安装完插件后重启 IDEA，到此 Lombok 插件就安装完成了，接下来就要进行实践演示了：

### Lombok 常用注解演示

首先在 pom 文件中引入依赖：

```
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.12</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

其中 `<scope>provided</scope>` 表示 jar 包是运行在编译时的，当程序编译成 class 源代码后，这个 jar 包就不会在源代码层面有所体现。

接下来演示 Lombok 注解使用方式，并通过查看编译后 class 文件，理解其工作原理，在这里以 `@Getter` 注解为例：

首先创建一个 GetterDemo 类，其中有 `name` 和 `age` 两个字段。

```
package com.wupx.lombok;

import lombok.AccessLevel;
import lombok.Getter;
import lombok.NonNull;

public class GetterDemo {

    @Getter(value = AccessLevel.PRIVATE, onMethod_ = {@NonNull})
    private String age;

    @Getter(lazy = true)
    private final String name = "wupx";
}
```

我们在变量 `age` 上加上注解 `@Getter`，并且加上了参数来设置访问级别，通过 `onMethod_` 参数可以为我们在生成的 `getAge` 方法添加上其他注解，比如 `@NonNull`；在 `name` 上加上 `@Getter` 注解，并加上 `lazy` 参数并设为 `true`，表示开启懒加载。

接下来编译下，编译的 class 源代码如下：

```
package com.wupx.lombok;

import java.util.concurrent.atomic.AtomicReference;
import lombok.NonNull;

public class GetterDemo {
    private String age;
    private final AtomicReference<Object> name = new AtomicReference();

    public GetterDemo() {
    }

    @NonNull
    private String getAge() {
        return this.age;
    }

    public String getName() {
        Object value = this.name.get();
        if (value == null) {
            synchronized(this.name) {
                value = this.name.get();
                if (value == null) {
                    String actualValue = "wupx";
                    value = "wupx" == null ? this.name : "wupx";
                    this.name.set(value);
                }
            }
        }

        return (String)((String)(value == this.name ? null : value));
    }
}
```

可以发现生成后的源代码文件中，`getAge` 方法访问修饰符为 `private`，并且方法上有一个 `@NonNull` 的注解；`getName` 方法没有刚开始就初始化一个字符串，而是只有调用该方法的时候判断该字段是否为空，若为空，则初始化一个字符串并返回，这样就可以为开销大的初始化操作做一个懒加载，只有当使用的时候才会主动加载这个字段。

其他的注解方法大家可以自己去实践操作下。

## Lombok 优缺点

在了解完 Lombok 之后，让我们来分析下 Lombok 的优缺点吧！

Lombok 的优点有以下几点：

- 通过注解自动生成样板代码，提高开发效率
- 代码简洁，只关注相关属性
- 新增属性后，无需刻意修改相关方法

但是 Lombok 也有一些缺点：

- 降低了源代码的可读性和完整性
- 加大对问题排查的难度（可能问题定位到不存在的行，无从下手）
- 强 x 队友，因为需要 IDE 的相关插件的支持

# 总结

本文介绍了什么是 Lombok，Lombok 做了什么以及 Lombok 的实现原理，并且分析了 Lombok 的利弊，大家在享受到它的好处的同时，也应该考虑到它带来的一些问题，你在工作中有被队友强 x 吗？你对 Lombok 怎么看？欢迎留言谈论！