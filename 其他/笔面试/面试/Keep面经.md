[TOC] 
# 一、Java
## 线程如何终止
1. 使用退出标志，使线程正常退出，也就是当run方法完成后线程终止。
2. 使用stop方法强行终止线程。
3. 使用interrupt方法中断线程。

## 如何用一个cancel方法停止两个线程
## 泛型原理、使用场景、优缺点
原理：泛型的实现是靠类型擦除技术，类型擦除是在编译期完成的，在编译期，编译器会将泛型的类型参数都擦除成它的限定类型，如果没有则擦除为object类型之后在获取的时候再强制类型转换为对应的类型。 

使用场景：参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口和泛型方法。

优点：
1. 类型安全
2. 消除强制类型转换
3. 潜在的性能收益

缺点：在性能上不如数组快。

## 手写代码，设计parseInt
```java
    public static int parseInt(String s) throws NumberFormatException {
        return parseInt(s,10);
    }
    public static int parseInt(String s, int radix)
                throws NumberFormatException
    {
        /*
         * 将整数字符串s转换成10进制的整数
         * radix用来指明s是几进制
         */
        //处理字符串s为空的情况
        if (s == null) {
            throw new NumberFormatException("null");
        }
        //Character.MIN_RADIX为2，就是进制数radix小于2进制的话也是无效的
        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }
        //Character.MAX_RADIX为36，就是进制数radix大于36进制的话也是无效的
        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        int result = 0;
        //判断正负号的标记，初始化为正
        boolean negative = false;
        int i = 0, len = s.length();
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;

        if (len > 0) {
            //取出第一个字符判断时候包含正负号
            char firstChar = s.charAt(0);
            if (firstChar < '0') {
                if (firstChar == '-') {
                    negative = true;
                    //若是字符串的符号是﹣，因为下面每次的result是相减的形式，所以这里是﹢的
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);

                if (len == 1) 
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;
            while (i < len) {
                // 返回使用指定radix进制的字符 s.charAt(i++) 的数值
                digit = Character.digit(s.charAt(i++),radix);
                // s.charAt(i++)的值是一个使用指定radix进制的无效数字，则返回 -1，异常
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                //上一次的结果乘以radix进制
                result *= radix;
                //处理溢出情况
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;
            }
        } else {//长度小于0的话，异常
            throw NumberFormatException.forInputString(s);
        }
        //negative是true的话，此整数是负的，输出result 
        //negative是false的话，此整数是正的，输出-result
        return negative ? result : -result;
    }
```

## hashmap是怎么实现的，是线程安全的吗
## 知道hashmap的扩容机制么
## arrylist实现原理
## 怎么实现线程安全
互斥同步：推荐使用 synchronized 关键字进行同步, 在 concurrent包中有ReentrantLock类, 实现效果差不多. 还是推荐原生态的synchronized.

非阻塞同步：需要硬件指令完成.常用的指令有:

Test-and-Set

Fetch-and-Increment

Swap

Compare-and-Swap (CAS)

Load-Linked/Store-Conditional (LL/SC)

典型的应用在 AtomicInteger 中

无同步方案：将变量保存在本地线程中，就不会出现多个线程并发的错误了。

java中主要使用的就是ThreadLocal这个类。

# 二、算法
## 从矩阵左上角到右下角的走法有多少种
## 一个长字符串，一个短字符串，短字符串中的字符间顺序我们可以任意改变，实现在长串中找到短串的代码
## Top k问题
## 求不相邻的最大子数组
## 排序算法有哪些？
## 介绍一下快排？
```
    private static void quickSort(int[] array, int _left, int _right) {
        int left = _left;//
        int right = _right;
        int pivot;//基准线
        if (left < right) {
            pivot = array[left];
            while (left != right) {
                //从右往左找到比基准线小的数
                while (left < right && pivot <= array[right]) {
                    right--;
                }
                //将右边比基准线小的数换到左边
                array[left] = array[right];
                //从左往右找到比基准线大的数
                while (left < right && pivot >= array[left]) {
                    left++;
                }
                //将左边比基准线大的数换到右边
                array[right] = array[left];
            }
            //此时left和right指向同一位置
            array[left] = pivot;
            quickSort(array, _left, left - 1);
            quickSort(array, left + 1, _right);
        }
    }
```

## 两个字符串找最长公共子串
## n个数中找到长度为m的和值最大的子串
## 归并思想

# 三、JVM
## 强软弱引用以及使用场景
## 对象的生命周期

## 如何判断对象能否回收
## 对象循环引用了怎么办
## 什么情况下会触发gc
## 内存泄漏有哪些场景、如何检测、如何避免
## java堆中存放的是什么，栈中存放什么。
## 类加载的过程
类加载的过程主要分为三个部分：
- 加载：指的是把class字节码文件从各个来源通过类加载器装载入内存中。
- 链接
- 初始化：对类变量初始化，是执行类构造器的过程。

链接又可以细分为
- 验证：为了保证加载进来的字节流符合虚拟机规范，不会造成安全错误。
- 准备：为类变量（注意，不是实例变量）分配内存，并且赋予初值。
- 解析：将常量池内的符号引用替换为直接引用的过程。


## jvm分区
![JVM内存模型](http://images.cnblogs.com/cnblogs_com/wupeixuan/1186116/o_JVM%e8%bf%90%e8%a1%8c%e6%97%b6%e6%95%b0%e6%8d%ae%e5%8c%ba.png)

**程序计数器**:记录正在执行的虚拟机字节码指令的地址（如果正在执行的是本地方法则为空）。

**Java虚拟机栈**:每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。

**本地方法栈**:与 Java 虚拟机栈类似，它们之间的区别只不过是本地方法栈为本地方法服务。

**Java堆**:几乎所有对象实例都在这里分配内存。是垃圾收集的主要区域（"GC 堆"），虚拟机把 Java 堆分成以下三块：
- 新生代
- 老年代
- 永久代

新生代又可细分为Eden空间、From Survivor空间、To Survivor空间，默认比例为8:1:1。

**方法区**：方法区（Method Area）与Java堆一样，是各个线程共享的内存区域。Object Class Data(类定义数据)是存储在方法区的，此外，常量、静态变量、JIT编译后的代码也存储在方法区。

**运行时常量池**：运行时常量池是方法区的一部分。Class 文件中的常量池（编译器生成的各种字面量和符号引用）会在类加载后被放入这个区域。除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()。这部分常量也会被放入运行时常量池。

**直接内存**：直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError 异常出现。避免在Java堆和Native堆中来回复制数据。

# 四、网络和数据库
## Mysql索引选择
## Mysql索引实现
## https原理
HTTPS(Secure Hypertext Transfer Protocol) 安全超文本传输协议是一个安全的通信通道，它基于HTTP开发，用于在客户计算机和服务器之间交换信息。HTTPS使用安全套接字层(SSL)进行信息交换，简单来说HTTPS是HTTP的安全版，是使用TLS/SSL加密的HTTP协议。

https通信过程
- 客户端发送请求到服务器端
- 服务器端返回证书和公开密钥，公开密钥作为证书的一部分而存在
- 客户端验证证书和公开密钥的有效性，如果有效，则生成共享密钥并使用公开密钥加密发送到服务器端
- 服务器端使用私有密钥解密数据，并使用收到的共享密钥加密数据，发送到客户端
- 客户端使用共享密钥解密数据
- SSL加密建立

# 五、操作系统
## 进程间通信有哪些方式
- 消息传递
    - 管道
    - 消息队列
    - 套接字
- 共享内存


# 六、设计模式
## 用过哪些设计模式
## 写线程安全的单例模式，为什么用volatile和synchronized，底层是怎么实现的，volatile是可重排序的吗
```
public class Singleton {
    private volatile static Singleton instance = null;

    private Singleton() {

    }

    /**
     * 当第一次调用getInstance()方法时，instance为空，同步操作，保证多线程实例唯一
     * 当第一次后调用getInstance()方法时，instance不为空，不进入同步代码块，减少了不必要的同步
     */
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

# 七、框架
## 介绍一下aop
AOP是对OOP的补充和完善。AOP利用的是代理，分为CGLIB动态代理和JDK动态代理。OOP引入封装、继承和多态性等概念来建立一种对象层次结构。OOP编程中，会有大量的重复代码。而AOP则是将这些与业务无关的重复代码抽取出来，然后再嵌入到业务代码当中。实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码，属于静态代理。

# 八、其他
## 设计一个微博 大v可能有几百万粉丝 大v发的微博关注他的用户会有实时通知 用户那里可以查看关注的所有人的微博
## 短域名和长域名。怎么根据短域名映射到对应的长域名，怎么存储，用什么数据结构。长域名怎么转化得到短域名的字符串？
## 统计一个网址访问次数前10多的ip地址。怎么保证实时性。