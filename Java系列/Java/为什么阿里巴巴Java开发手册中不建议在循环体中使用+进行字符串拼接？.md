之前在阅读《阿里巴巴Java开发手册》时，发现有一条是关于循环体中字符串拼接的建议，具体内容如下：

![阿里巴巴Java开发手册](https://img-blog.csdnimg.cn/20191023192646912.png)

那么我们首先来用例子来看看在循环体中用 + 或者用 StringBuilder 进行字符串拼接的效率如何吧（JDK版本为 jdk1.8.0_201）。

```
public class StringConcatDemo {
    public static void main(String[] args) {
        long s1 = System.currentTimeMillis();
        new StringConcatDemo().addMethod();
        System.out.println("使用 + 拼接:" + (System.currentTimeMillis() - s1));

        s1 = System.currentTimeMillis();
        new StringConcatDemo().stringBuilderMethod();
        System.out.println("使用 StringBuilder 拼接:" + (System.currentTimeMillis() - s1));
    }

    public String addMethod() {
        String result = "";
        for (int i = 0; i < 100000; i++) {
            result += (i + "wupx");
        }
        return result;
    }

    public String stringBuilderMethod() {
        StringBuilder result = new StringBuilder();
        for (int i = 0; i < 100000; i++) {
            result.append(i).append("wupx");
        }
        return result.toString();
    }
}
```

执行结果如下：

```
使用 + 拼接:29282
使用 StringBuilder 拼接:4
```

为什么这两种方法的时间会差这么多呢？接下来让我们一起进一步研究。

# 为什么 StringBuilder 比 + 快这么多？
从字节码层面来看下，为什么循环体中字符串拼接 StringBuilder 比 + 快这么多？

使用 javac StringConcatDemo.java 命令编译源文件，使用 javap -c StringConcatDemo 命令查看字节码文件的内容。

其中 addMethod() 方法的字节码如下：

```
public java.lang.String addMethod();
Code:
   0: ldc           #16                 // String
   2: astore_1
   3: iconst_0
   4: istore_2
   5: iload_2
   6: ldc           #17                 // int 100000
   8: if_icmpge     41
  11: new           #7                  // class java/lang/StringBuilder
  14: dup
  15: invokespecial #8                  // Method java/lang/StringBuilder."<init>":()V
  18: aload_1
  19: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  22: iload_2
  23: invokevirtual #18                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
  26: ldc           #19                 // String wupx
  28: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  31: invokevirtual #12                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
  34: astore_1
  35: iinc          2, 1
  38: goto          5
  41: aload_1
  42: areturn
```

可以看出，第 8 行到第 38 行构成了一个循环体：在第 8 行的时候做条件判断，如果不满足循环条件，则跳转到 41 行。编译器做了一定程度的优化，在 11 行 new 了一个 StringBuilder 对象，然后再 19 行、23 行、28 行进行了三次 append() 方法的调用，不过每次循环都会重新 new 一个 StringBuilder 对象。

再来看 stringBuilderMethod() 方法的字节码：

```
public java.lang.String stringBuilderMethod();
Code:
   0: new           #7                  // class java/lang/StringBuilder
   3: dup
   4: invokespecial #8                  // Method java/lang/StringBuilder."<init>":()V
   7: astore_1
   8: iconst_0
   9: istore_2
  10: iload_2
  11: ldc           #17                 // int 100000
  13: if_icmpge     33
  16: aload_1
  17: iload_2
  18: invokevirtual #18                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
  21: ldc           #19                 // String wupx
  23: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  26: pop
  27: iinc          2, 1
  30: goto          10
  33: aload_1
  34: invokevirtual #12                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
  37: areturn
```
13 行到 30 行构成了循环体，可以看出，在第4行（循环体外）就构建好了 StringBuilder 对象，然后再循环体内只进行 append() 方法的调用。

由此可以看出，在 for 循环中，使用 + 进行字符串拼接，每次都是 new 了一个 StringBuilder，然后再把 String 转成 StringBuilder，再进行 append，而频繁的新建对象不仅要耗费很多时间，还会造成内存资源的浪费。这就从字节码层面解释了为什么不建议在循环体内使用 + 去进行字符串的拼接。

接下来再来让我们看下使用 + 或者 StringBuilder 拼接字符串的原理吧。

# 使用 + 拼接字符串
在 Java 开发中，最简单常用的字符串拼接方法就是直接使用 + 来完成：

```
String boy = "wupx";
String girl = "huxy";
String love = boy + girl;
```

反编译后的内容如下：（使用的反编译工具为 jad）

```
String boy = "wupx";
String girl = "huxy";
String love = (new StringBuilder()).append(boy).append(girl).toString();
```

通过查看反编译以后的代码，可以发现，在字符串常量在拼接过程中，是将 String 转成了 StringBuilder 后，使用其 append() 方法进行处理的。

那么也就是说，Java中的 + 对字符串的拼接，其实现原理是使用 StringBuilder 的 append() 来实现的，使用 + 拼接字符串，其实只是 Java 提供的一个语法糖。

# 使用 StringBuilder 拼接字符串
StringBuilder 的 append 方法就是第二个常用的字符串拼接姿势了。

和 String 类类似，StringBuilder 类也封装了一个字符数组，定义如下：
```
char[] value;
```

与 String 不同的是，它并不是 final 的，所以是可以修改的。另外，与 String 不同，字符数组中不一定所有位置都已经被使用，它有一个实例变量，表示数组中已经使用的字符个数，定义如下：
```
int count;
```

其 append() 方法源码如下：
```
public StringBuilder append(String str) {
   super.append(str);
   return this;
}
```

该类继承了 AbstractStringBuilder 类，看下其 append() 方法：

```
public AbstractStringBuilder append(String str) {
    if (str == null)
        return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
}
```

首先判断拼接的字符串 str 是不是 null，如果是，调用 appendNull() 方法进行处理，appendNull() 方法的源码如下：

```
private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
```

如果字符串 str 不为 null，则判断拼接后的字符数组长度是否超过当前数组长度，如果超过，则调用 Arrays.copyOf() 方法进行扩容并复制，ensureCapacityInternal() 方法的源码如下：

```
private void ensureCapacityInternal(int minimumCapacity) {
    if (minimumCapacity - value.length > 0) {
        value = Arrays.copyOf(value,
                newCapacity(minimumCapacity));
    }
}
```

最后，将拼接的字符串 str 复制到目标数组 value 中。

```
str.getChars(0, len, value, count);
```

# 总结
本文针对《阿里巴巴Java开发手册》中的循环体中拼接字符串建议出发，从字节码层面，来解释为什么 StringBuilder 比 + 快，还分别介绍了字符串拼接中 + 和 StringBuilder 的原理，因此在循环体拼接字符串时，应该使用 StringBuilder 的 append() 去完成拼接。

> 参考
> 
> 《Java开发手册》华山版
>
> 《Effective Java（第二版）》
> 
> 《Java编程思想》