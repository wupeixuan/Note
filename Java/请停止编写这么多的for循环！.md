在这篇文章中，我想和你一起回到基础知识，并讨论 Java 中的 for 循环。老实说，我正在为自己写这篇博客文章，因为我也会这样做。从 Java 8 开始，我们不必在 Java 中编写太多 for 循环！我希望这篇文章将使你的代码更易于阅读和编写。

# 你需要 for 循环做什么？

一般地说，for 循环执行两类任务:

- 遍历集合
- 运行算法

对于算法，for 循环可能是合适的。看一下此算法，检查数字是否为三的幂：

```
double number = 81;
for(; number > 1; number /=3);
return number == 1;
```

在这里，for 循环是合适的。这是一个非常简单的示例，你可以想象，使用更困难的算法会变得更加棘手。

对于大多数开发人员而言，在他们的日常工作中，这种情况很少。大多数时候，我们使用 for 循环遍历集合。让我们看一下该代码的一些示例。

# 遍历 Java 中的集合

我们首先来定义一个 List<String> 数组，并往里面插入一些元素：

```
List<String> heroes = new ArrayList<>();
heroes.add("典韦");
heroes.add("云中君");
heroes.add("鲁班");
heroes.add("盘古");
heroes.add("大乔");
heroes.add("宫本武藏");
heroes.add("裴擒虎");
heroes.add("明世隐");
```

有很多方法可以对其进行迭代。让我们从相当古老的迭代器方法开始：

```
Iterator<String> heroesIterator = heroes.iterator();
while (heroesIterator.hasNext()) {
    System.out.println(heroesIterator.next());
}
```

这种代码使 Java 在冗长性上享有应有的声誉。


这次尝试使用经典的索引 for 循环：

```
for (int i = 0; i < heroes.size(); i++) {
    System.out.println(heroes.get(i));
}
```

嗯，这有点简单，但是在 Java 5 之后，我们可以这样处理循环：

```
for (String hero : heroes) {
    System.out.println(hero);
}
```

这是大多数开发人员陷入困境的地方。这种结构非常熟悉并且易于遵循，以至于我们大多数人都不会去考虑更好的东西。但是在 Java 8 以后我们可以使用 forEach 函数来进行简化。

借助 Java 8，我们可以使用 forEach 函数：

```
heroes.forEach(hero -> System.out.println(hero));
```

我们可以进一步简化它：

```
heroes.forEach(System.out::println);
```

在我们只是迭代数组的元素时，这样会使代码很简洁。

# 下一步呢？使用 Java Streams

一旦停止在 Java 中编写如此多的 for 循环，forEach 就成为了你的第二选择，那么你应该看看 Java 中的 Streams。

例如，使用类似的语法，你可以轻松选择所有以 “鲁” 开头的英雄：

```
heroes.stream().filter(hero -> hero.startsWith("鲁"))
        .forEach(System.out::println);
```

运行结果会显示小短腿“鲁班”。

# 总结
停止编写太多 for 循环，完成后，Java 8 Streams 将自然而然地出现，你的代码将更易于阅读和编写。