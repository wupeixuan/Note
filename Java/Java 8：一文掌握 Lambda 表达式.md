本文将介绍 Java 8 新增的 Lambda 表达式，包括 Lambda 表达式的常见用法以及方法引用的用法，并对 Lambda 表达式的原理进行分析，最后对 Lambda 表达式的优缺点进行一个总结。


![image](https://img-blog.csdnimg.cn/20191023203210650.png)

# 1. 概述
Java 8 引入的 Lambda 表达式的主要作用就是简化部分匿名内部类的写法。

能够使用 Lambda 表达式的一个重要依据是必须有相应的函数接口。所谓函数接口，是指内部有且仅有一个抽象方法的接口。

Lambda 表达式的另一个依据是类型推断机制。在上下文信息足够的情况下，编译器可以推断出参数表的类型，而不需要显式指名。

# 2. 常见用法
## 2.1 无参函数的简写
无参函数就是没有参数的函数，例如 Runnable 接口的 run() 方法，其定义如下：

```
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

在 Java 7 及之前版本，我们一般可以这样使用：

```
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
        System.out.println("Jimmy");
    }
}).start();
```

从 Java 8 开始，无参函数的匿名内部类可以简写成如下方式：

```
() -> {
    执行语句
}
```

这样接口名和函数名就可以省掉了。那么，上面的示例可以简写成：

```
new Thread(() -> {
    System.out.println("Hello");
    System.out.println("Jimmy");
}).start();
```

当只有一条语句时，我们还可以对代码块进行简写，格式如下：

```
() -> 表达式
```

注意这里使用的是表达式，并不是语句，也就是说不需要在末尾加分号。

那么，当上面的例子中执行的语句只有一条时，可以简写成这样：

```
new Thread(() -> System.out.println("Hello")).start();
```

## 2.2 单参函数的简写
单参函数是指只有一个参数的函数。例如 View 内部的接口 OnClickListener 的方法 onClick(View v)，其定义如下：

```
public interface OnClickListener {
    /**
     * Called when a view has been clicked.
     *
     * @param v The view that was clicked.
     */
    void onClick(View v);
}
```

在 Java 7 及之前的版本，我们通常可能会这么使用：

```
view.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        v.setVisibility(View.GONE);
    }
});
```

从 Java 8 开始，单参函数的匿名内部类可以简写成如下方式：

```
([类名 ]变量名) -> {
    执行语句
}
```

其中类名是可以省略的，因为 Lambda 表达式可以自己推断出来。那么上面的例子可以简写成如下两种方式：


```
view.setOnClickListener((View v) -> {
    v.setVisibility(View.GONE);
});
view.setOnClickListener((v) -> {
    v.setVisibility(View.GONE);
});
```

单参函数甚至可以把括号去掉，官方也更建议使用这种方式：

```
变量名 -> {
    执行语句
}
```

那么，上面的示例可以简写成：

```
view.setOnClickListener(v -> {
    v.setVisibility(View.GONE);
});
```

当只有一条语句时，依然可以对代码块进行简写，格式如下：

```
([类名 ]变量名) -> 表达式
```


类名和括号依然可以省略，如下：

```
变量名 -> 表达式
```

那么，上面的示例可以进一步简写成：


```
view.setOnClickListener(v -> v.setVisibility(View.GONE));
```

## 2.3 多参函数的简写
多参函数是指具有两个及以上参数的函数。例如，Comparator 接口的 compare(T o1, T o2) 方法就具有两个参数，其定义如下：

```
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

在 Java 7 及之前的版本，当我们对一个集合进行排序时，通常可以这么写：

```
List<Integer> list = Arrays.asList(1, 2, 3);
Collections.sort(list, new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return o1.compareTo(o2);
    }
});
```

从 Java 8 开始，多参函数的匿名内部类可以简写成如下方式：

```
([类名1 ]变量名1, [类名2 ]变量名2[, ...]) -> {
    执行语句
}
```

同样类名可以省略，那么上面的例子可以简写成：

```
Collections.sort(list, (Integer o1, Integer o2) -> {
    return o1.compareTo(o2);
});
Collections.sort(list, (o1, o2) -> {
    return o1.compareTo(o2);
});
```

当只有一条语句时，依然可以对代码块进行简写，格式如下：

```
([类名1 ]变量名1, [类名2 ]变量名2[, ...]) -> 表达式
```

此时类名也是可以省略的，但括号不能省略。如果这条语句需要返回值，那么 return 关键字是不需要写的。

因此，上面的示例可以进一步简写成：

```
Collections.sort(list, (o1, o2) -> o1.compareTo(o2));
```

最后呢，这个示例还可以简写成这样：

```
Collections.sort(list, Integer::compareTo);
```

咦，这是什么特性？这就是我们下面要讲的内容：方法引用。

# 3. 方法引用
方法引用也是一个语法糖，可以用来简化开发。

在我们使用 Lambda 表达式的时候，如果“->”的右边要执行的表达式只是调用一个类已有的方法，那么就可以用「方法引用」来替代 Lambda 表达式。

方法引用可以分为 4 类：

- 引用静态方法；
- 引用对象的方法；
- 引用类的方法；
- 引用构造方法。

下面按照这 4 类分别进行阐述。

## 3.1 引用静态方法
当我们要执行的表达式是调用某个类的静态方法，并且这个静态方法的参数列表和接口里抽象函数的参数列表一一对应时，我们可以采用引用静态方法的格式。

假如 Lambda 表达式符合如下格式：

```
([变量1, 变量2, ...]) -> 类名.静态方法名([变量1, 变量2, ...])
```

我们可以简写成如下格式：

```
类名::静态方法名
```

注意这里静态方法名后面不需要加括号，也不用加参数，因为编译器都可以推断出来。下面我们继续使用 2.3 节的示例来进行说明。

首先创建一个工具类，代码如下：

```
public class Utils {
    public static int compare(Integer o1, Integer o2) {
        return o1.compareTo(o2);
    }
}
```

注意这里的 compare() 函数的参数和 Comparable 接口的 compare() 函数的参数是一一对应的。然后一般的 Lambda 表达式可以这样写：

```
Collections.sort(list, (o1, o2) -> Utils.compare(o1, o2));
```

如果采用方法引用的方式，可以简写成这样：

```
Collections.sort(list, Utils::compare);
```

## 3.2 引用对象的方法
当我们要执行的表达式是调用某个对象的方法，并且这个方法的参数列表和接口里抽象函数的参数列表一一对应时，我们就可以采用引用对象的方法的格式。

假如 Lambda 表达式符合如下格式：

```
([变量1, 变量2, ...]) -> 对象引用.方法名([变量1, 变量2, ...])
```

我们可以简写成如下格式：

```
对象引用::方法名
```

下面我们继续使用 2.3 节的示例来进行说明。首先创建一个类，代码如下：

```
public class MyClass {
    public int compare(Integer o1, Integer o2) {
        return o1.compareTo(o2);
    }
}
```

当我们创建一个该类的对象，并在 Lambda 表达式中使用该对象的方法时，一般可以这么写：

```
MyClass myClass = new MyClass();
Collections.sort(list, (o1, o2) -> myClass.compare(o1, o2));
```

注意这里函数的参数也是一一对应的，那么采用方法引用的方式，可以这样简写：

```
MyClass myClass = new MyClass();
Collections.sort(list, myClass::compare);
```

此外，当我们要执行的表达式是调用 Lambda 表达式所在的类的方法时，我们还可以采用如下格式：

```
this::方法名
```

例如我在 Lambda 表达式所在的类添加如下方法：

```
private int compare(Integer o1, Integer o2) {
    return o1.compareTo(o2);
}
```

当 Lambda 表达式使用这个方法时，一般可以这样写：

```
Collections.sort(list, (o1, o2) -> compare(o1, o2));
```

如果采用方法引用的方式，就可以简写成这样：

```
Collections.sort(list, this::compare);
```

## 3.3 引用类的方法
引用类的方法所采用的参数对应形式与上两种略有不同。如果 Lambda 表达式的“->”的右边要执行的表达式是调用的“->”的左边第一个参数的某个实例方法，并且从第二个参数开始（或无参）对应到该实例方法的参数列表时，就可以使用这种方法。

可能有点绕，假如我们的 Lambda 表达式符合如下格式：

```
(变量1[, 变量2, ...]) -> 变量1.实例方法([变量2, ...])
```

那么我们的代码就可以简写成：

```
变量1对应的类名::实例方法名
```

还是使用 2.3 节的例子， 当我们使用的 Lambda 表达式是这样时：

```
Collections.sort(list, (o1, o2) -> o1.compareTo(o2));
```

按照上面的说法，就可以简写成这样：

```
Collections.sort(list, (o1, o2) -> o1.compareTo(o2));
```

## 3.4 引用构造方法
当我们要执行的表达式是新建一个对象，并且这个对象的构造方法的参数列表和接口里函数的参数列表一一对应时，我们就可以采用「引用构造方法」的格式。

假如我们的 Lambda 表达式符合如下格式：

```
([变量1, 变量2, ...]) -> new 类名([变量1, 变量2, ...])
```

我们就可以简写成如下格式：

```
类名::new
```

下面举个例子说明一下。Java 8 引入了一个 Function 接口，它是一个函数接口，部分代码如下：

```
@FunctionalInterface
public interface Function<T, R> {
    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
		// 省略部分代码
}
```

我们用这个接口来实现一个功能，创建一个指定大小的 ArrayList。一般我们可以这样实现：

```
Function<Integer, ArrayList> function = new Function<Integer, ArrayList>() {
    @Override
    public ArrayList apply(Integer n) {
        return new ArrayList(n);
    }
};
List list = function.apply(10);
```

使用 Lambda 表达式，我们一般可以这样写：

```
Function<Integer, ArrayList> function = n -> new ArrayList(n);
```

使用「引用构造方法」的方式，我们可以简写成这样：

```
Function<Integer, ArrayList> function = ArrayList::new;
```

## 4. 自定义函数接口
自定义函数接口很容易，只需要编写一个只有一个抽象方法的接口即可，示例代码：

```
@FunctionalInterface
public interface MyInterface<T> {
    void function(T t);
}
```

上面代码中的 @FunctionalInterface 是可选的，但加上该注解编译器会帮你检查接口是否符合函数接口规范。就像加入 @Override 注解会检查是否重写了函数一样。

# 5. 实现原理
经过上面的介绍，我们看到 Lambda 表达式只是为了简化匿名内部类书写，看起来似乎在编译阶段把所有的 Lambda 表达式替换成匿名内部类就可以了。但实际情况并非如此，在 JVM 层面，Lambda 表达式和匿名内部类其实有着明显的差别。

## 5.1 匿名内部类的实现
匿名内部类仍然是一个类，只是不需要我们显式指定类名，编译器会自动为该类取名。比如有如下形式的代码：

```
public class LambdaTest {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello World");
            }
        }).start();
    }
}
```

编译之后将会产生两个 class 文件：

```
LambdaTest.class
LambdaTest$1.class
```

使用 javap -c LambdaTest.class 进一步分析 LambdaTest.class 的字节码，部分结果如下：

```
public static void main(java.lang.String[]);
  Code:
     0: new           #2                  // class java/lang/Thread
     3: dup
     4: new           #3                  // class com/example/myapplication/lambda/LambdaTest$1
     7: dup
     8: invokespecial #4                  // Method com/example/myapplication/lambda/LambdaTest$1."<init>":()V
    11: invokespecial #5                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
    14: invokevirtual #6                  // Method java/lang/Thread.start:()V
    17: return
```

可以发现在 4: new #3 这一行创建了匿名内部类的对象。

## 5.2 Lambda 表达式的实现
接下来我们将上面的示例代码使用 Lambda 表达式实现，代码如下：

```
public class LambdaTest {
    public static void main(String[] args) {
        new Thread(() -> System.out.println("Hello World")).start();
    }
}
```

此时编译后只会产生一个文件 LambdaTest.class，再来看看通过 javap 对该文件反编译后的结果：

```
public static void main(java.lang.String[]);
  Code:
     0: new           #2                  // class java/lang/Thread
     3: dup
     4: invokedynamic #3,  0              // InvokeDynamic #0:run:()Ljava/lang/Runnable;
     9: invokespecial #4                  // Method java/lang/Thread."<init>":(Ljava/lang/Runnable;)V
    12: invokevirtual #5                  // Method java/lang/Thread.start:()V
    15: return
```

从上面的结果我们发现 Lambda 表达式被封装成了主类的一个私有方法，并通过 invokedynamic 指令进行调用。

因此，我们可以得出结论：Lambda 表达式是通过 invokedynamic 指令实现的，并且书写 Lambda 表达式不会产生新的类。

既然 Lambda 表达式不会创建匿名内部类，那么在 Lambda 表达式中使用 this 关键字时，其指向的是外部类的引用。

# 6. 优缺点
优点：

- 可以减少代码的书写，减少匿名内部类的创建，节省内存占用。
- 使用时不用去记忆所使用的接口和抽象函数。

缺点：

- 易读性较差，阅读代码的人需要熟悉 Lambda 表达式和抽象函数中参数的类型。
- 不方便进行调试。