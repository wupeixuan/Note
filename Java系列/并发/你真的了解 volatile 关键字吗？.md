今天，让我们一起来探讨 Java 并发编程中的知识点：volatile 关键字

本文主要从以下三点讲解 volatile 关键字：
1. volatile 关键字是什么？
2. volatile 关键字能解决什么问题？使用场景是什么？
3. volatile 关键字实现的原理？

# volatile 关键字是什么？

在 Sun 的 JDK 官方文档是这样形容 volatile 的：
> The Java programming language provides a second mechanism, volatile fields, that is more convenient than locking for some purposes. A field may be declared volatile, in which case the Java Memory Model ensures that all threads see a consistent value for the variable.

也就是说，如果一个变量加了 volatile 关键字，就会告诉编译器和 JVM 的内存模型：这个变量是对所有线程共享的、可见的，每次 JVM 都会读取最新写入的值并使其最新值在所有 CPU 可见。volatile 可以保证线程的可见性并且提供了一定的有序性，但是无法保证原子性。在 JVM 底层 volatile 是采用内存屏障来实现的。
 
通过这段话，我们可以知道 volatile 有两个特性：
1. 保证可见性、不保证原子性
2. 禁止指令重排序

# 原子性和可见性
原子性是指一个操作或多个操作要么全部执行并且执行的过程不会被任何因素打断，要么都不执行。性质和数据库中事务一样，一组操作要么都成功，要么都失败。看下面几个简单例子来理解原子性：
```
i == 0;       //1
j = i;        //2
i++;          //3
i = j + 1;    //4
```

在看答案之前，可以先思考一下上面四个操作，哪些是原子操作？哪些是非原子操作？

答案揭晓：
```
1——是：在Java中，对基本数据类型的变量赋值操作都是原子性操作（Java 有八大基本数据类型，分别是byte，short，int，long，char，float，double，boolean）
2——不是：包含两个动作：读取 i 值，将 i 值赋值给 j
3——不是：包含了三个动作：读取 i 值，i+1，将 i+1 结果赋值给 i
4——不是：包含了三个动作：读取 j 值，j+1，将 j+1 结果赋值给 i
```

也就是说，只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。

注：由于以前的操作系统是 32 位， 64 位数据（long 型,double 型）在 Java 中是 8 个字节表示，一共占用 64 位，因此需要分成两次操作采用完成一个变量的赋值或者读取操作。随着 64 位操作系统越来越普及，在 64 位的 HotSpot JVM 实现中，对64 位数据（long 型,double 型）做原子性处理（由于 JVM 规范没有明确规定，不排除别的 JVM 实现还是按照 32 位的方式处理）。

在单线程环境中我们可以认为上述步骤都是原子性操作，但是在多线程环境下，Java 只保证了上述基本数据类型的赋值操作是原子性的，其他操作都有可能在运算过程中出现错误。为此在多线程环境下为了保证一些操作的原子性引入了锁和 synchronized 等关键字。

上面说到 volatile 关键字保证了变量的可见性，不保证原子性。原子性已经说了，下面说下可见性。

可见性其实和 Java 内存模型的设定有关：Java 内存模型规定所有的变量都是存在主存（线程共享区域）当中，每个线程都有自己的工作内存（私有内存）。线程对变量的所有操作都必须在工作内存中进行，而不直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。

举个简单栗子：

比如上面 i++ 操作，在 Java 中，执行 `i++` 语句：

执行线程首先从主存中读取 i（原始值）到工作内存中，然后在工作内存中执行运算 +1 操作（主存的 i 值未变），最后将运算结果刷新到主存中。

数据运算是在执行线程的私有内存中进行的，线程执行完运算后，并不一定会立即将运算结果刷新到主存中（虽然最后一定会更新主存），刷新到主存动作是由 CPU 自行选择一个合适的时间触发的。假设数值未更新到主存之前，当其他线程去读取时(而且优先读取的是工作内存中的数据而非主存)，此时主存中可能还是原来的旧值，就有可能导致运算结果出错。
 
以下代码是测试代码：
```
package com.wupx.test;

/**
 * @author wupx
 * @date 2019/10/31
 */
public class VolatileTest {

    private boolean flag = false;

    class ThreadOne implements Runnable {
        @Override
        public void run() {
            while (!flag) {
                System.out.println("执行操作");
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("任务停止");
        }
    }

    class ThreadTwo implements Runnable {
        @Override
        public void run() {
            try {
                Thread.sleep(2000L);
                System.out.println("flag 状态改变");
                flag = true;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        VolatileTest testVolatile = new VolatileTest();
        Thread thread1 = new Thread(testVolatile.new ThreadOne());
        Thread thread2 = new Thread(testVolatile.new ThreadTwo());
        thread1.start();
        thread2.start();
    }
}
``` 
上述结果有可能在线程 2 执行完 flag = true 之后，并不能保证线程 1 中的 while 能立即停止循环，原因在于 flag 状态首先是在线程 2 的私有内存中改变的，刷新到主存的时机不固定，而且线程 1 读取 flag 的值也是在自己的私有内存中，而线程 1 的私有内存中 flag 仍未 false，这样就有可能导致线程仍然会继续 while 循环。运行结果如下：

```
执行操作
执行操作
执行操作
flag 状态改变
任务停止
```

避免上述不可预知问题的发生就是用 volatile 关键字修饰 flag，volatile 修饰的共享变量可以保证修改的值会在操作后立即更新到主存里面，当有其他线程需要操作该变量时，不是从私有内存中读取，而是强制从主存中读取新值。即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。

# 指令重排序
一般来说，处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。
 
比如下面的代码
```
int i = 0;             
boolean flag = false;
i = 1;        // 1
flag = true;  // 2
```

代码定义了一个 int 型变量，定义了一个 boolean 类型变量，然后分别对两个变量进行赋值操作。从代码顺序上看，语句 1 是在语句 2 前面的，那么 JVM 在真正执行这段代码的时候会保证语句 1 一定会在语句 2 前面执行吗？不一定，为什么呢？这里可能会发生指令重排序（InstructionReorder）。

语句 1 和语句 2 谁先执行对最终的程序结果并没有影响，那么就有可能在执行过程中，语句 2 先执行而语句 1 后执行。

但是要注意，虽然处理器会对指令进行重排序，但是它会保证程序最终结果会和代码顺序执行结果相同，那么它靠什么保证的呢？再看下面一个例子：
 
```
int a = 10;     // 1
int r = 2;      // 2
a = a + 3;      // 3
r = a * a;      // 4
```

这段代码执行的顺序可能是 1->2->3->4 或者是 2->1->3->4，但是 3 和 4 的执行顺序是不会变的，因为处理器在进行重排序时是会考虑指令之间的数据依赖性，如果一个指令 Instruction2 必须用到 Instruction1 的结果，那么处理器会保证 Instruction1 会在 Instruction2 之前执行。

虽然重排序不会影响单个线程内程序执行的结果，但是多线程呢？下面看一个例子：

```
// 线程1
String config = initConfig();    // 1
boolean inited = true;           // 2
 
// 线程2
while(!inited){
       sleep();
}
doSomeThingWithConfig(config);
```

上面代码中，由于语句 1 和语句 2 没有数据依赖性，因此可能会被重排序。假如发生了重排序，在线程 1 执行过程中先执行语句 2，而此时线程 2 会以为初始化工作已经完成，那么就会跳出 while 循环，去执行 doSomeThingWithConfig(config) 方法，而此时 config 并没有被初始化，就会导致程序出错。
 
从上面可以看出，指令重排序不会影响单个线程的执行，但是会影响到线程并发执行的正确性。

那么 volatile 关键字修饰的变量禁止重排序的含义是：

1. 当程序执行到 volatile 变量的读操作或者写操作时，在其前面的操作肯定已经全部进行，且对后面的操作可见，在其后面的操作肯定还没有进行
2. 在进行指令优化时，不能将 volatile 变量之前的语句放在对 volatile 变量的读写操作之后，也不能把 volatile 变量后面的语句放到其前面执行

举个栗子：
```
x=0;             // 1
y=1;             // 2
volatile z = 2;  // 3
x=4;             // 4
y=5;             // 5
```
变量z为 volatile 变量，那么进行指令重排序时，不会将语句 3 放到语句 1、语句 2 之前，也不会将语句 3 放到语句 4、语句 5 后面。但是语句 1 和语句 2、语句 4 和语句 5 之间的顺序是不作任何保证的，并且 volatile 关键字能保证，执行到语句 3 时，语句 1 和语句 2 必定是执行完毕了的，且语句 1 和语句 2 的执行结果是对语句 3、语句 4、语句 5是可见的。

回到之前的例子：
```
// 线程1
String config = initConfig();   // 1
volatile boolean inited = true; // 2
 
// 线程2
while(!inited){
       sleep();
}
 
doSomeThingWithConfig(config);
```
之前说这个例子提到有可能语句2会在语句1之前执行，那么就可能导致执行 doSomThingWithConfig() 方法时就会导致出错。

这里如果用 volatile 关键字对 inited 变量进行修饰，则可以保证在执行语句 2 时，必定能保证 config 已经初始化完毕。
 
# volatile 应用场景
synchronized 关键字是防止多个线程同时执行一段代码，那么就会很影响程序执行效率，而 volatile 关键字在某些情况下性能要优于 synchronized，但是要注意 volatile 关键字是无法替代 synchronized 关键字的，因为 volatile 关键字无法保证操作的原子性。通常来说，使用 volatile 必须具备以下三个条件：
1. 对变量的写入操作不依赖变量的当前值，或者能确保只有单个线程更新变量的值
2. 该变量不会与其他状态变量一起纳入不变性条件中
3. 在访问变量时不需要加锁

上面的三个条件只需要保证是原子性操作，才能保证使用 volatile 关键字的程序在高并发时能够正确执行。建议不要将 volatile 用在 getAndOperate 场合，仅仅 set 或者 get 的场景是适合 volatile 的。
 
常用的两个场景是：

1. 状态标记量

```
volatile boolean flag = false;

while (!flag) {
    doSomething();
}

public void setFlag () {
    flag = true;
}

volatile boolean inited = false;
// 线程 1
context = loadContext();
inited = true;

// 线程 2
while (!inited) {
    sleep();
}
doSomethingwithconfig(context);
```

2. DCL 双重校验锁-单例模式

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

推荐阅读：[设计模式-单例模式](https://blog.csdn.net/qq_27276045/article/details/88895177)

使用 volatile 的原因在上面解释重排序时已经讲过了。主要在于 instance = new Singleton()，这并非是一个原子操作，在 JVM 中这句话做了三件事情：
1. 给 instance分配内存
2. 调用 Singleton 的构造函数来初始化成员变量
3. 将 instance 对象指向分配的内存库存空间（执行完这步 instance 就为非 null 了）

但是 JVM 即时编译器中存在指令重排序的优化，也就是说上面的第二步和第三步顺序是不能保证的，最终的执行顺序可能是 1-2-3，也可能是 1-3-2。如果是后者，线程 1 在执行完 3 之后，2 之前，被线程 2 抢占，这时 instance 已经是非 null（但是并没有进行初始化），所以线程 2 返回 instance 使用就会报空指针异常。

# volatile 特性是如何实现的呢？
前面讲述了关于 volatile 关键字的一些使用，下面我们来探讨一下 volatile 到底如何保证可见性和禁止指令重排序的。

在《深入理解Java虚拟机》这本书中说道：

> 观察加入volatile关键字和没有加入 volatile 关键字时所生成的汇编代码发现，加入 volatile 关键字时，会多出一个 lock 前缀指令。

接下来举个栗子：

volatile 的 Integer 自增（i++），其实要分成 3 步：
1. 读取 volatile 变量值到 local
2. 增加变量的值
3. 把 local 的值写回，让其它的线程可见
 
这 3 步的 JVM 指令为：
```
mov    0xc(%r10),%r8d ; Load
inc    %r8d           ; Increment
mov    %r8d,0xc(%r10) ; Store
lock addl $0x0,(%rsp) ; StoreLoad Barrier
```
lock 前缀指令实际上相当于一个内存屏障（也叫内存栅栏），内存屏障会提供 3 个功能：
 
1. 它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成（满足禁止重排序）
2. 它会强制将对缓存的修改操作立即写入主存（满足可见性）
3. 如果是写操作，它会导致其他 CPU 中对应的缓存行无效（满足可见性）

volatile 变量规则是 happens-before(先行发生原则)中的一种：对一个变量的写操作先行发生于后面对这个变量的读操作。（该特性可以很好解释 DCL 双重检查锁单例模式为什么使用 volatile 关键字来修饰能保证并发安全性）

# 总结
变量声明为 volatile 类型时，编译器与运行时都会注意到这个变量是共享的，不会将该变量上的操作与其他内存操作一起重排序。volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取 volatile 类型的变量时总会返回最新写入的值。

在访问 volatile 变量时不会执行加锁操作，也就不会使执行线程阻塞，因此 volatile 变量是比 sychronized 关键字更轻量级的同步机制。

加锁机制既可以确保可见性和原子性，而 volatile 变量只能确保可见性。

> 参考
> 
> 《深入理解Java虚拟机》