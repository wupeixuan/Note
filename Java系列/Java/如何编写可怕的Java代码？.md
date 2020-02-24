我决定告诉你如何编写可怕的Java代码。如果你厌倦了所有这些美丽的设计模式和最佳实践，并且想写些疯狂的东西，请继续阅读。

如果你正在寻找有关如何编写良好代码的建议，请查看其它文章！

# 对一切使用异常
你知道循环对吗？差一错误（英语：Off-by-one error，缩写 OBOE，是在计数时由于边界条件判断失误导致结果多了一或少了一的错误，通常指计算机编程中循环多了一次或者少了一次的程序错误，属于逻辑错误的一种）很容易犯。当你迭代一个集合时，很容易出错。让我们看看如何使用 Java 异常处理来解决该问题，而不用担心这些讨厌的差一错误！

```
public static void horribleIteration(String [] words){
    int i = 0;
    try {
        while(true){
            System.out.println(words[i]);
            i++;
        }
    } catch (IndexOutOfBoundsException e){
        //iteration complete
    }
}
```

# 不用担心访问修饰符

你说什么？Java 中的访问修饰符，这不是浪费时间嘛！你是否知道将属性/方法等设为私有只是一个建议？如果你想修改它，那就去做吧！没什么能阻止你（除了缺乏知识之外）。如果是这种情况，请看如下代码。

```
public static void readPrivate() throws NoSuchFieldException, IllegalAccessException {
    Field f = System.class.getDeclaredField("lineSeparator");
    f.setAccessible(true);
    String separator = (String) f.get(System.class);
    System.out.println("Line separator is " + separator + ".");
}
```

我们在这里读取 lineSeparator，这并没有什么。但是修改 lineSeparator 会带来更多乐趣！在我们修改代码中的 lineSeparator 之后，看看 System.out.println 发生了什么：

```
public static void readWritePrivate() throws NoSuchFieldException, IllegalAccessException {
    Field f = System.class.getDeclaredField("lineSeparator");
    f.setAccessible(true);
    String separator = (String) f.get(System.class);
    System.out.println("Line separator is " + separator + ".");
 
    f.set(System.class ,"!!!");
    System.out.println("Line one");
    System.out.println("Line two");
    System.out.println("Line three");
}
```

输出为：

```
Line separator is 
WARNING: All illegal access operations will be denied in a future release
.
Line one!!!Line two!!!Line three!!!
```

看起来不错!

# 在 Java 中没有什么是真正的 final

一些开发人员认为他们通过将 final 关键字放在变量前面来以说明不会去更改这个值。事实是——有时候你真的想要改变一个 final 字段的值，所以这是如何做的：

```
public static void notSoFinal() throws NoSuchFieldException, IllegalAccessException, InterruptedException {
    ExampleClass example = new ExampleClass(10);
    System.out.println("Final value was: "+ example.finalValue);
    Field f = example.getClass().getDeclaredField("finalValue");
    Field modifiersField = Field.class.getDeclaredField("modifiers");
    modifiersField.setAccessible(true);
    modifiersField.setInt(f, f.getModifiers() & ~Modifier.FINAL);
    f.setInt(example, 77);
    System.out.println("Final value was: "+ example.finalValue);
}
 
public static class ExampleClass {
    final int finalValue;
 
    public ExampleClass(int finalValue){
        this.finalValue = finalValue;
    }
}
```

注意，在构造函数中提供最终值时，这对我很有用。如果你在类中设置了 final 值，那么它将不起作用。（可能是一些编译器级别的优化破坏了所有的乐趣）

# 使用 Java 序列化，干就对了

这很简单，用 Java 序列化，玩得开心，好好享受。

好吧，我想你想要一些理由。我看到 Java 平台首席架构师 Mark Reinhold 表示，他们后悔将序列化引入到 Java。显然，Java 中大约 1/3 的安全漏洞仅来自于序列化。

# 将对象用于一切

你知道类吗？浪费时间！你是否想看到代码重用的巅峰之作？你去！

```
public static void printThings (List things){
    int i = 0;
    try {
        while(true){
            System.out.println(things.get(i));
            i++;
        }
    } catch (IndexOutOfBoundsException e){
        //iteration complete
    }
}
 
List superList = new ArrayList();
superList.add(7);
superList.add("word");
superList.add(true);
superList.add(System.class);
printThings(superList);
```

您可以相信我们一直以来都拥有这种力量吗？另外，组合两个模式还有额外的好处!

这只是你使用 Object 进行操作的开始。如果有疑问，请记住-使用对象。如果需要，你随时可以使用这种惊人的模式进行回退！

```
public static void printThingsUppercaseStrings (List things){
    int i = 0;
    try {
        while(true){
            Object o = things.get(i);
            System.out.println(o);
            if(o.getClass() == String.class){
                String so = (String) o;
                so = so.toUpperCase();
                System.out.println(so);
            }
            i++;
        }
    } catch (IndexOutOfBoundsException e){
        //iteration complete
    }
}
```

这还是类型安全的，多么健壮的解决方案。

# 充分拥抱便捷编程的艺术

你知道比尔·盖茨更喜欢懒惰的开发人员吗？比尔·盖茨说过：

> "I will always choose a lazy person to do a difficult job...because, he will find an easy way to do it. --Bill Gates" 
> 
> "我总是会选择一个懒人去完成一份困难的工作...因为，他会找到捷径。" -- 比尔盖茨

因此，有了比尔·盖茨（Bill Gates）的大力支持，我们可以完全接受我们的懒惰。你准备好了吗？那就开始吧！

- 永远不要编写测试，只是不要编写错误！
- 将所有都定义为 public -方便访问！
- 支持全局变量–您可能需要它们！
- 大型接口优于小型专用接口–可以使用的方法越多越好！
- 支持继承而不是合成（使用接口中的默认方法从未如此简单）！
- 始终使用装箱类型–它们也可以用作对象！
- 尽可能使用最短的名字(a, b, n 最好)！

# 不要学习任何新知识–你总是最了解

一个程序员最重要的品质就是对自己有信心。相信自己什么都懂，没有什么可学的！

考虑到这一点，请确保不要学习：

- 新类库
- 新语言
- 新框架

这样可以节省你的时间！你永远都不应学习任何新知识，因为你已经是最好的了。

> 你有能力去做这件事,并不代表你应该做