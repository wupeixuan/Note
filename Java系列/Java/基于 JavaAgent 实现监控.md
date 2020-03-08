# 前言介绍

全链路监控又名分布式监控系统全链路追踪，目前市面的全链路监控系统基本都是参考Google的Dapper（大规模分布式系统的跟踪系统）来做的。例如；蚂蚁金服分布式链路跟踪组件SOFATracer、Gokit微服务-服务链路追踪 、Pinpoint、Prometheus(普罗米修斯)等等。

# 案例简述

JavaAgent是在JDK5之后提供的新特性，也可以叫java代理。开发者通过这种机制(Instrumentation)可以在加载class文件之前修改方法的字节码(此时字节码尚未加入JVM)，动态更改类方法实现AOP，提供监控服务如；方法调用时长、可用率、内存等。本章节初步怎么让java代码执行时可以进入我们的agent方法。

# 实践

工程目录结构如下：

```
agent-demo
├─ package
│	└─ agent-demo.jar
├─ pom.xml
├─ src
│	├─ main
│	│	├─ java
│	│	└─ resources
│	└─ test
│	 	└─ java
```

首先用 IDEA 创建一个 maven 项目，然后新建一个类 `MyAgent`，内容如下：

```
package com.wupx.agent;

import java.lang.instrument.Instrumentation;

/**
 * @author wupx
 * @date 2020/2/28
 */
public class MyAgent {
    
    public static void premain(String agentArgs, Instrumentation inst) {
        System.out.println("MyAgent premain run");
    }

}
```

然后创建一个测试类 `TestAgent`，具体内容如下：

```
/**
 * @author wupx
 * @date 2020/2/28
 */
public class TestAgent {
    public static void main(String[] args) {
        System.out.println("TestAgent main run");
    }
}
```

因为Java Agent的特殊性，需要一些特殊的配置，在META-INF目录下创建 MANIFEST 文件，建议创建在 resources 目录下，文件中需要指定 Agent 的启动类，具体内容如下：

```
Manifest-Version: 1.0
Premain-Class: com.wupx.agent.MyAgent
Archiver-Version: Plexus Archiver
Built-By: wupx
Can-Redefine-Classes: true
Created-By: Apache Maven 3.6.1
Build-Jdk: 1.8.0_151

```

为什么需要指定 Premain-Class 呢？因为在加载 Java Agent 之后，会找到 Premain-Class 指定的类，并运行对应的 premain 方法。


如果不想手动创建 MANIFEST 文件，也可以通过 Maven 配置，在打包的时候自动生成，具体配置参数如下：

```
<build>
    <finalName>agent-demo</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <archive>
                    <manifestEntries>
                        <Premain-Class>com.wupx.agent.MyAgent</Premain-Class>
                        <Can-Redefine-Classes>true</Can-Redefine-Classes>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后需要修改 VM options 的参数为 `-javaagent:D:\Program\workspace\agent-demo\package\agent-demo.jar` （大家需要根据自己的实际路径修改）

然后运行 `TestAgent` 类，会发现运行结果如下：

```
MyAgent premain run
TestAgent main run

Process finished with exit code 0
```

说明和我们讲的一样，程序启动的时候，会优先加载 Java Agent，并执行其 premain 方法。

到此为止，我们已经知道通过配置JVM `-javaagent:**.jar` 后，在java程序启动时候会执行premain方法。接下来我们使用javassist字节码增强的方式，来监控方法程序的执行耗时。

Javassist是一个开源的分析、编辑和创建Java字节码的类库。是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建的。它已加入了开放源代码JBoss应用服务器项目，通过使用Javassist对字节码操作为JBoss实现动态"AOP"框架。

关于java字节码的处理，目前有很多工具，如bcel，asm。不过这些都需要直接跟虚拟机指令打交道。如果你不想了解虚拟机指令，可以采用javassist。javassist是jboss的一个子项目，其主要的优点，在于简单，而且快速。直接使用java编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构，或者动态生成类。

