这篇文章主要来介绍下 JDK 内置的命令工具，话不多说，让我们开始吧！

首先登场的是 `javap`：

## javap

使用 `javap` 可以查看 Java 字节码反编译的源文件，`javap` 的命令格式如下：

```
-help  --help  -?        输出此用法消息
-version                 版本信息
-v  -verbose             输出附加信息
-l                       输出行号和本地变量表
-public                  仅显示公共类和成员
-protected               显示受保护的/公共类和成员
-package                 显示程序包/受保护的/公共类
                        和成员 (默认)
-p  -private             显示所有类和成员
-c                       对代码进行反汇编
-s                       输出内部类型签名
-sysinfo                 显示正在处理的类的
                        系统信息 (路径, 大小, 日期, MD5 散列)
-constants               显示最终常量
-classpath <path>        指定查找用户类文件的位置
-cp <path>               指定查找用户类文件的位置
-bootclasspath <path>    覆盖引导类文件的位置
```

下面来演示下用 `javap -c` 对代码进行反编译，首先写个 `HelloWorld` 类，如下：

```
public class HelloWorld {
    public static void main(String []args) {
       System.out.println("Hello World");
    }
}
```

接着使用 `javap -c HelloWorld.class` 就可以反编译得到如下结果：

```
Compiled from "HelloWorld.java"
public class HelloWorld {
  public HelloWorld();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #3                  // String Hello World
       5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}
```