final 是 Java 中重要关键字之一，可以应用于类、方法以及变量上。这篇文章中将讲解什么是 final 关键字？将变量、方法和类声明为 final 代表了什么？使用 final 的好处是什么？

# final 关键字是什么？
final 在 Java 中是一个保留的关键字，可以声明成员变量、方法、类以及本地变量。一旦你将引用声明作 final，你将不能改变这个引用了，编译器会检查代码，如果试图将变量再次初始化的话，编译器会报编译错误。

# final 变量
凡是对成员变量或者本地变量(在方法中的或者代码块中的变量称为本地变量)声明为 final 的都叫作 final 变量。final 变量经常和 static 关键字一起使用，作为常量。下面是 final 变量的例子：

```
public static final String NAME = "wupx";
NAME = new String("wupx"); //invalid compilation error
```

final 变量是只读的。

# final 方法
final 也可以声明方法，Java 里用 final 修饰符去修饰一个方法的唯一正确用途就是表达：这个方法原本是一个虚方法，现在通过 final 来声明这个方法不允许在派生类中进一步被覆写（override）。

Java 中非私有的成员方法默认都是虚方法，而虚方法就可以在派生类中被覆写。为了保证某个类上的某个虚方法不在派生类中被进一步覆写，就需要使用 final 修饰符来声明，让编译器（例如 javac）与 JVM 共同检查并保证这个限制总是成立。

下面引用 R 大 在知乎上的回答来打破“用 final 修饰方法可以让对这个方法的调用变快”的流言：


> 曾经有一种广为流传的说法是用final修饰方法可以让对这个方法的调用变快。这种说法在现代主流的优化JVM上都是不成立的（例如Oracle JDK / OpenJDK HotSpot VM、IBM J9 VM、Azul Systems Zing VM等）。这是因为：能用final修饰的虚方法，其派生类中必然不可能存在对其覆写的版本，于是可以判定这个虚方法只有一个可能的调用目标；而如果此时把这个final修饰符去掉，这些先进的JVM都可以通过“类层次分析”（Class Hierarchy Analysis，CHA）来发现这个方法在派生类中没有进一步覆写的版本，于是同样可以判定这个虚方法只有一个可能的调用目标。然后两者的优化程度会一模一样，无论是从“不需要通过虚分派而可以直接调用目标”（称为“去虚化”，devirtualization）还是从“便于内联”的角度看，这两种情况都是一样的。
> 
> 而如果某个类层次结构中原本某个虚方法就存在多个覆写版本的话，那么本来也不可能对这个虚方法加上final修饰，所以就算这种情况下去虚化变得困难，锅也不能让“因为没用final修饰”来背。
> 
> 使用final关键字在现代主流的优化JVM上不会提升性能。


下面是 final 方法的例子：

```
class User{
    public final String getName(){
        return "user：wupx";
    }
}

class Reader extends User{
    @Override
    public final String getName(){
        return "reader wupx"; //compilation error: overridden method is final
    }
}
```

# final 类
使用 final 来修饰的类叫作 final 类，final类通常功能是完整的，它们不能被继承，Java 中有许多类是 final 的，比如 String, Interger 以及其他包装类。下面是 final 类的实例：

```
final class User{

}

class Reader extends User{  //compilation error: cannot inherit from final class

}
```

# 内存模型中的 final
对于 final 变量，编译器和处理器都要遵守两个重排序规则：

- 构造函数内，对一个 final 变量的写入，与随后把这个被构造对象的引用赋值给一个变量，这两个操作之间不可重排序
- 首次读一个包含 final 变量的对象，与随后首次读这个 final 变量，这两个操作之间不可以重排序

实际上这两个规则也正是针对 final 变量的写与读。写的重排序规则可以保证，在对象引用对任意线程可见之前，对象的 final 变量已经正确初始化了，而普通变量则不具有这个保障；读的重排序规则可以保证，在读一个对象的 final 变量之前，一定会先读这个对象的引用。如果读取到的引用不为空，根据上面的写规则，说明对象的 final 变量一定以及初始化完毕，从而可以读到正确的变量值。

如果 final 变量的类型是引用型，那么构造函数内，对一个 final 引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。

实际上这也是为了保证 final 变量在对其他线程可见之前，能够正确的初始化完成。

# final 关键字的好处
下面为使用 final 关键字的一些好处：

- final 关键字提高了性能，JVM 和 Java 应用都会缓存 final 变量
- final 变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销

# 总结
- final 关键字可以用于成员变量、本地变量、方法以及类
- final 成员变量必须在声明的时候初始化或者在构造器中初始化，否则就汇报编译错误
- 不能够对 final 变量再次赋值
- 本地变量必须在声明时赋值
- 在匿名类中所有变量都必须是 final 变量
- final 方法不能被重写
- final 类不能被继承
- 接口中声明的所有变量本身是 final 的
- final 和 abstract 这两个关键字是反相关的，final 类就不可能是 abstract 的
- 没有在声明时初始化 final 变量的称为空白 final 变量(blank final variable)，它们必须在构造器中初始化，或者调用 this() 初始化，不这么做的话，编译器会报错final变量(变量名)需要进行初始化
- 按照 Java 代码惯例，final 变量就是常量，而且通常常量名要大写
- 对于集合对象声明为 final 指的是引用不能被更改


> 参考
> 
> 《Java编程思想》
> 
> https://www.zhihu.com/question/66083114