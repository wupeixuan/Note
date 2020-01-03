Java 中 final、finally、finalize 有什么不同？这是在 Java 面试中经常问到的问题，他们究竟有什么不同呢？

这三个看起来很相似，其实他们的关系就像卡巴斯基和巴基斯坦一样有基巴关系。

那么如果被问到这个问题该怎么回答呢？首先可以从语法和使用角度出发简单介绍三者的不同：

- final 可以用来修饰类、方法、变量，分别有不同的意义，final 修饰的 class 代表不可以继承扩展，比如 String 类；final 的变量是不可以修改的；Java 里用 final 修饰符去修饰一个方法的唯一正确用途就是表达：这个方法原本是一个虚方法，现在通过 final 来声明这个方法不允许在派生类中进一步被覆写（override）。
- finally 是 Java 保证重点代码一定要被执行的一种机制。可以使用 try-finally 或者 try-catch-finally 来进行关闭资源、保证 unlock 锁等动作。
- finalize 是基础类 java.lang.Object 的一个方法，设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize 机制现在已经不推荐使用，并且在 JDK 9 开始被标记为 deprecated。

如果只回答到这里，就会没有亮点，我们可以再深入地去介绍三者的不同。

# final
使用 final 关键字可以明确表示代码的语义、逻辑意图，比如：

可以将方法或者类声明为 final，这样就可以明确告知别人，这些行为是不许修改的。
Java 核心类库的定义或源码，比如 java.lang 包下面的很多类，相当一部分都被声明成为 final class，比如我们常见的 String 类，在第三方类库的一些基础类中同样如此，这可以有效避免 API 使用者更改基础功能，某种程度上，这是保证平台安全的必要手段。

使用 final 修饰参数或者变量，也可以清楚地避免意外赋值导致的编程错误，甚至，有人明确推荐将所有方法参数、本地变量、成员变量声明成 final。

final 变量产生了某种程度的不可变（immutable）的效果，所以，可以用于保护只读数据，尤其是在并发编程中，因为明确地不能再赋值 final 变量，有利于减少额外的同步开销，也可以省去一些防御性拷贝的必要。

关于 final 也许会有性能的好处，很多文章或者书籍中都介绍了可在特定场景提高性能，比如，利用 final 可能有助于 JVM 将方法进行内联，可以改善编译器进行条件编译的能力等等。我在之前一篇文章进行了介绍，想了解的可以点击查阅。

## final 与 immutable
在前面介绍了 final 在实践中的益处，需要注意的是，final 并不等同于 immutable，比如下面这段代码：
```
final List<String> strList = new ArrayList<>();
strList.add("wupx");
strList.add("huxy");  
List<String> loveList = List.of("wupx", "huxy");
loveList.add("love");
```

final 只能约束 strList 这个引用不可以被赋值，但是 strList 对象行为不被 final 影响，添加元素等操作是完全正常的。如果我们真的希望对象本身是不可变的，那么需要相应的类支持不可变的行为。在上面这个例子中，List.of 方法创建的本身就是不可变 List，最后那句 add 是会在运行时抛出异常的。

Immutable 在很多场景是非常棒的选择，某种意义上说，Java 语言目前并没有原生的不可变支持，如果要实现 immutable 的类，我们需要做到：

将 class 自身声明为 final，这样别人就不能扩展来绕过限制了。

将所有成员变量定义为 private 和 final，并且不要实现 setter 方法。

通常构造对象时，成员变量使用深度拷贝来初始化，而不是直接赋值，这是一种防御措施，因为你无法确定输入对象不被其他人修改。

如果确实需要实现 getter 方法，或者其他可能会返回内部状态的方法，使用 copy-on-write 原则，创建私有的 copy。

关于 setter/getter 方法，很多人喜欢直接用 IDE 或者 Lombok 一次全部生成，建议最好确定有需要时再实现。

# finally
对于 finally，知道怎么使用就足够了。需要关闭的连接等资源，更推荐使用 Java 7 中添加的 try-with-resources 语句，因为通常 Java 平台能够更好地处理异常情况，还可以减少代码量。

另外，有一些常被考到的 finally 问题。比如，下面代码会输出什么？

```
try {
  // do something
  System.exit(1);
} finally{
  System.out.println("Hello，I am finally。");
}
```

上面 finally 里面的代码是不会被执行的，因为 try-catch 异常退出了。

像其他 finally 中的代码不会执行的情况还有：

```
// 死循环
try{
    while(ture){
        System.out.println("always run");
    }
}finally{
    System.out.println("ummm");
}

// 线程被杀死
当执行 try-finally 的线程被杀死时，finally 中的代码也无法执行。
```

# finalize
对于 finalize，是不推荐使用的，在 Java 9 中，已经将 Object.finalize() 标记为 deprecated。

为什么呢？因为无法保证 finalize 什么时候执行，执行的是否符合预期。使用不当会影响性能，导致程序死锁、挂起等。

通常来说，利用上面的提到的 try-with-resources 或者 try-finally 机制，是非常好的回收资源的办法。如果确实需要额外处理，可以考虑 Java 提供的 Cleaner 机制或者其他替代方法。

## 为什么不推荐使用 finalize？
前面简单介绍了 finalize 是不推荐使用的，究竟为什么不推荐使用呢？

1. finalize 的执行是和垃圾收集关联在一起的，一旦实现了非空的 finalize 方法，就会导致相应对象回收呈现数量级上的变慢。
2. finalize 被设计成在对象被垃圾收集前调用，JVM 要对它进行额外处理。finalize 本质上成为了快速回收的阻碍者，可能导致对象经过多个垃圾收集周期才能被回收。
3. finalize 拖慢垃圾收集，导致大量对象堆积，也是一种典型的导致 OOM 的原因。
4. 要确保回收资源就是因为资源都是有限的，垃圾收集时间的不可预测，可能会极大加剧资源占用。
5. finalize 会掩盖资源回收时的出错信息。

因此对于消耗非常高频的资源，千万不要指望 finalize 去承担资源释放的主要职责。建议资源用完即显式释放，或者利用资源池来尽量重用。

下面给出 finalize 掩盖资源回收时的出错信息的例子，让我们来看 java.lang.ref.Finalizer 的源代码：

```
private void runFinalizer(JavaLangAccess jla) {
    //  ... 省略部分代码
    try {
        Object finalizee = this.get(); 
        if (finalizee != null && !(finalizee instanceof java.lang.Enum)) {
           jla.invokeFinalize(finalizee);
           // Clear stack slot containing this variable, to decrease
           // the chances of false retention with a conservative GC
           finalizee = null;
        }
    } catch (Throwable x) { }
        super.clear(); 
}
```

看过之前讲解异常文章的朋友，应该可以很快看出 Throwable 是被吞掉的，也就意味着一旦出现异常或者出错，得不到任何有效信息。

## 有更好的方法替代 finalize 吗？

Java 平台目前在逐步使用 java.lang.ref.Cleaner 来替换掉原有的 finalize 实现。Cleaner 的实现利用了幻象引用（PhantomReference），这是一种常见的所谓 post-mortem 清理机制。利用幻象引用和引用队列，可以保证对象被彻底销毁前做一些类似资源回收的工作，比如关闭文件描述符（操作系统有限的资源），它比 finalize 更加轻量、更加可靠。

每个 Cleaner 的操作都是独立的，有自己的运行线程，所以可以避免意外死锁等问题。

我们可以为自己的模块构建一个 Cleaner，然后实现相应的清理逻辑，具体代码如下：

```
/**
 * Cleaner 是一个用于关闭资源的类，功能类似 finalize 方法
 * Cleaner 有自己的线程，在所有清理操作完成后，自己会被 GC
 * 清理中抛出的异常会被忽略
 * 
 * 清理方法（一个 Runnable）只会运行一次。会在两种情况下运行：
 * 1. 注册的 Object 处于幻象引用状态
 * 2. 显式调用 clean 方法
 * 
 * 通过幻象引用和引用队列实现
 * 可以注册多个对象，通常被定义为静态（减少线程数量）
 * 注册对象后返回的Cleanable对象用于显式调用 clean 方法
 * 实现清理行为的对象（下面的 state），不能拥有被清理对象的引用
 * 如果将下面的 State 类改为非静态，第二个 CleaningExample 将不会被 clean，
 * 因为非静态内部类持有外部对象的引用，外部对象无法进入幻象引用状态
 */
public class CleaningExample implements AutoCloseable {

    public static void main(String[] args) {
        try {
            // 使用JDK7的try with Resources显式调用clean方法
            try (CleaningExample ignored = new CleaningExample()) {
                throw new RuntimeException();
            }
        } catch (RuntimeException ignored) {
        }

        // 通过GC调用clean方法
        new CleaningExample();
        System.gc();
    }

    private static final Cleaner CLEANER = Cleaner.create();

    // 如果是非静态内部类，则会出错
    static class State implements Runnable {
        State() {
        }

        @Override
        public void run() {
            System.out.println("Cleaning called");
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    public CleaningExample() {
        this.state = new State();
        this.cleanable = CLEANER.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }

}
```
其中，将 State 定义为 static，就是为了避免普通的内部类隐含着对外部对象的强引用，因为那样会使外部对象无法进入幻象可达的状态。

从可预测性的角度来判断，Cleaner 或者幻象引用改善的程度仍然是有限的，如果由于种种原因导致幻象引用堆积，同样会出现问题。所以，Cleaner 适合作为一种最后的保证手段，而不是完全依赖 Cleaner 进行资源回收。

# 总结
这篇文章首先从从语法角度分析了 final、finally、finalize，并从安全、性能、垃圾收集等方面逐步深入，详细地讲解了 final、finally、finalize 三者的区别。

> 参考
> 
> https://blog.csdn.net/qq_27276045/article/details/102774119
> 
> https://blog.csdn.net/qq_27276045/article/details/102762241
> 
> https://time.geekbang.org/column/article/6906