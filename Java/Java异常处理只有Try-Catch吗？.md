今天，我们将讨论一个非常重要的主题-Java 中的异常处理。尽管有时可能会对此主题进行过多的讨论，但并非每篇文章都包含有用且相关的信息。

Java 中最常见的异常处理机制通常与 try-catch 块关联  。我们使用它来捕获异常，然后提供在发生异常的情况下可以执行的逻辑。

的确，你不需要将所有异常都放在这些块中。另一方面，如果你正在研究应用程序的软件设计，则可能不需要内置的异常处理机制。在这种情况下，你可以尝试使用替代方法-Vavr Try 结构。

在本文中，我们将探讨 Java 异常处理的不同方法，并讨论如何使用 Vavr Try 替代内置方法。让我们开始吧！

# 处理 Java 中的异常
作为介绍，让我们回顾一下 Java 如何允许我们处理异常。如果你不记得它，则 Java 中的异常会指出意外或意外事件，该异常在程序执行期间（即在运行时）发生，这会破坏程序指令的正常流程。Java为我们提供了上述 try-catch 捕获异常的机制。让我们简要检查一下它是如何工作的。

## 如果不处理异常会发生什么？

首先，让我们看一个非常常见的例子。这是一个包含 JDBC 代码的代码段：

```
Connection connection = dataSource.getConnection();
String updateNameSql = "UPDATE employees SET name=? WHERE emp_id=?";
PreparedStatement preparedStatement = connection.prepareStatement(updateNameSql);
```

坦白地说，你的 IDE 甚至不允许你编写这段代码，而是要求用 try-catch 块将其包围，像这样：

```
try {
    Connection connection = dataSource.getConnection();
    String updateNameSql = "UPDATE employees SET name=? WHERE emp_id=?";
    PreparedStatement preparedStatement = connection.prepareStatement(updateNameSql);
} catch (SQLException ex){
}
```

注：我们可以将其重构为 try-with-resources，但是稍后再讨论。

那么，为什么我们要这样编写代码？因为 SQLException 是一个检查异常。

如果这些异常可以由方法或构造函数的执行抛出并传播到方法或构造函数边界之外，则必须在方法或构造函数的 throws 子句中声明这些异常。SQLException 如果发生数据库访问错误，则在示例中使用的方法将抛出  。因此，我们用一个 try-catch 块将其包围。

Java 在编译过程中验证了这些异常，这就是它们与运行时异常不同的原因。

## 但是你不必处理所有异常情况

但是，并非每个异常都应被一个 try-catch 块包围。

### 情况 1：运行时异常

Java 异常是 Throwable 的子类，但是其中一些是 RuntimeException 类的子类。看下面的图，它给出了 Java 异常的层次结构：

![](https://img-blog.csdnimg.cn/201912011506511.png)

请注意，运行时异常是特定的组。根据 Java 规范，从这些异常中还是有可能恢复的。作为示例，让我们回想一下 ArrayIndexOutOfBoundsException。看看下面的示例代码片段：

```
int numbers[] = [1,43,51,0,9];
System.out.println(numbers[6]);
```

在这里，我们有一个具有5个值（0-4位）的整数数组。当我们尝试检索绝对超出范围的值（索引= 6）时，Java 将抛出 ArrayIndexOutOfBoundsException。

这表明我们尝试调用的索引为负数，大于或等于数组的大小。如我所说，这些异常可以修复，因此在编译过程中不会对其进行检查。这意味着你仍然可以编写如下代码：

```
int numbers[] = [1,43,51,0,9];
int index = 6;
try{
    System.out.println(numbers[index]);
} catch (ArrayIndexOutOfBoundsException ex){
    System.out.println("Incorrect index!");
}
```

但是你不必这样做。


### 情况 2：错误

Error 是另一个棘手的概念。再看一下上面的图-存在错误，但是通常不会处理。为什么？通常，这是由于 Java 程序无法执行任何操作来从错误中恢复，例如：错误表明严重的问题，而合理的应用程序甚至不应尝试捕获。

让我们考虑一下内存错误– java.lang.VirtualMachineError。此错误表明 JVM 已损坏或已经用尽了继续运行所必需的资源。换句话说，如果应用程序的内存不足，则它根本无法分配额外的内存资源。

当然，如果由于持有大量应释放的内存而导致失败，则异常处理程序可以尝试释放它（不是直接释放它本身，而是可以调用JVM来释放它）。并且，尽管这样的处理程序在这种情况下可能有用，但是这样的尝试可能不会成功。

## Try-Catch 块的变体

上述编写  try-catch 语句的方法并不是 Java 中唯一可用的方法。还有其他方法：try-with-resources，try-catch-finally 和多个 catch 块。让我们快速浏览这些不同的方法。


### 方法 1：Try-With-Resources

try-with-resources 块在 Java 7 中引入的，并允许开发者在程序运行到此结束后必须关闭声明的资源。我们可以在实现该 AutoCloseable 接口（即特定标记接口）的任何类中包含资源。我们可以像这样重构所提到的 JDBC 代码：

```
try (Connection connection = dataSource.getConnection){
    String updateNameSql = "UPDATE employees SET name=? WHERE emp_id=?";
    PreparedStatement preparedStatement = connection.prepareStatement(updateNameSql);
} catch (SQLException ex){
//..
}
```

Java 确保我们 Connection 在执行代码后将其关闭。在进行此构建之前，我们必须显式地关闭 finally 块中的资源。

### 方法 2：Try + Finally

finally 块在任何情况下都将执行。例如在成功情况下或在异常情况下。在其中，你需要放置将在之后执行的代码：

```
FileReader reader = null;
try {
    reader = new FileReader("/text.txt");
    int i=0;
    while(i != -1){
        i = reader.read();
        System.out.println((char) i);
    }
} catch(IOException ex1){
//...
} finally{
    if(reader != null){
       try {
         reader.close();
       } catch (IOException ex2) {
       //...
       }
    }
}
```

请注意，此方法有一个缺点：如果在 finally 块内引发异常  ，则会使其中断。因此，我们必须正常处理异常。将 try-with-resources 与可关闭的资源一起使用，避免在 finally 块内关闭资源 。


### 方法 3：多 Catch 块

最后，Java 允许我们使用一个 try-catch 块多次捕获异常。当方法抛出几种类型的异常并且您想区分每种情况的逻辑时，这很有用。举个例子，让这个虚构的类使用抛出多个异常的方法：


```
class Repository{
    void insert(Car car) throws DatabaseAccessException, InvalidInputException {
    //...
    }
}
//...
try {
    repository.insert(car);
} catch (DatabaseAccessException dae){
    System.out.println("Database is down!");
} catch (InvalidInputException iie){
    System.out.println("Invalid format of car!");
}
```

在这里需要记住什么？通常，我们假设在此代码中，这些异常处于同一级别。但是你必须从最具体到最一般的顺序排序 catch 块。例如，捕获 ArithmeticException 异常必须在 捕获 Exception 异常之前。

到这里，我们已经回顾了如何使用内置方法处理 Java 中的异常。现在，让我们看一下如何使用 Vavr 库执行此操作。

# Vavr Try
我们回顾了捕获 Java 异常的标准方法。另一种方法是使用 Vavr Try 类，Vavr 是 Java 8+ 中一个函数式库，提供了一些不可变数据类型及函数式控制结构。首先，添加 Vavr 库依赖项：

```
<dependency>
   <groupId>io.vavr</groupId>
   <artifactId>vavr</artifactId>
   <version>0.10.2</version>
</dependency>
```

## Try 容器
Vavr 包含的 Try 类是 monadic 容器类型，它表示可能导致异常或返回成功计算出的值的计算。此结果可以采用 Success 或 Failure。看下面这段代码：

```
class CarsRepository{
    Car insert(Car car) throws DatabaseAccessException {
        //...
    }
    Car find (String id) throws DatabaseAccessException {
        //..
    }
    void update (Car car) throws DatabaseAccessException {
        //..
    }
    void remove (String id) throws DatabaseAccessException {
        //..
    }
}
```

在调用此代码时，我们将使用这些 try-catch 块来处理 DatabaseAccessException。但是另一个解决方案是使用 Vavr 对其进行重构。查看以下代码片段：


```
class CarsVavrRepository{
    Try<Car> insert(Car car) {
        System.out.println("Insert a car...");
        return Try.success(car);
    }
    Try<Car> find (String id) {
        System.out.println("Finding a car...");
        System.out.println("..something wrong with database!");
        return Try.failure(new DatabaseAccessException());
    }
    Try<Car> update (Car car) {
        System.out.println("Updating a car...");
        return Try.success(car);
    }
    Try<Void> remove (String id) {
        System.out.println("Removing a car...");
        System.out.println("..something wrong with database!");
        return Try.failure(new DatabaseAccessException());
    }
}
```

现在，我们可以使用 Vavr 处理数据库问题。

## 处理成功
当我们收到成功计算的结果时，我们会收到 Success：

```
@Test
void successTest(){
    CarsVavrRepository repository = new CarsVavrRepository();
    Car skoda = new Car("skoda", "9T4 4242", "black");
    Car result = repository.insert(skoda).getOrElse(new Car("volkswagen", "3E2 1222", "red"));
    Assertions.assertEquals(skoda.getColor(), result.getColor());
    Assertions.assertEquals(skoda.getId(), result.getId());
}
```

请注意，Vavr.Try 相较于 Vavr.Option，为我们提供了一种方便的 getOrElse 方法，在发生故障的情况下我们可以使用默认值，我们可以将这种逻辑与有问题的方法结合使用，例如与 find 一起使用。

## 处理失败
在另一种情况下，我们将处理 Failure：

```
@Test
void failureTest(){
    CarsVavrRepository repository = new CarsVavrRepository();
        // desired car
    Car bmw = new Car("bmw", "4A1 2019", "white");
        // failure car
    Car failureCar = new Car("seat", "1A1 3112", "yellow");
    Car result = repository.find("4A1 2019").getOrElse(failureCar);
    Assertions.assertEquals(bmw.getColor(), result.getColor());
    Assertions.assertEquals(bmw.getId(), result.getId());
}

```

运行此代码。由于断言错误，该测试将失败：

```
org.opentest4j.AssertionFailedError: 
Expected :white
Actual   :yellow
```

这意味着因为我们在 find 方法中对失败进行了硬编码  ，所以我们收到了默认值。除了返回默认值之外，我们还可以在发生错误的情况下执行其他操作并生成结果。你可以使用链接函数 Option 来使您的代码更具功能性：

```
repository.insert(bmw).andThen(car -> {
    System.out.println("Car is found "+car.getId());
}).andFinally(()->{
    System.out.println("Finishing");
});
```

或者，你可以使用收到的异常执行代码，如下所示：


```

repository.find("1A9 4312").orElseRun(error->{
    //...
});
```

一般来说，Vavr Try 是一个功能丰富的解决方案，可用于以更实用的方式转换代码库。毫无疑问，它与其他 Vavr 类（如 Option 或 Collections）结合后，才可以释放出真正的力量。但是， 如果您想编写更多的功能样式的代码，即使没有它们，Vavr Try 对于 Java 的 try-catch 块来说也是一个的正确的替代选择。

# 总结
Java 中的异常处理机制通常与 try-catch 块关联，  以便捕获异常并提供发生异常时将要执行的逻辑。同样，我们确实不需要将所有异常都放入这些块中。在本文中，我们探讨了如何使用 Vavr 库执行此操作。