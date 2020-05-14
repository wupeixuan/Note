本篇文章将详细介绍 Optional 类，以及如何用它消除代码中的 null 检查。在开始之前首先来看下什么是 NPE，以及在 Java 8 之前是如何处理 NPE 问题的。

空指针异常（NullPointException，简称 NPE）可以说是所有 Java 程序员都遇到过的一个异常，虽然 Java 从设计之初就力图让程序员脱离指针的苦海，但是指针确实是实际存在的，而 Java 设计者也只能是让指针在 Java 语言中变得更加简单易用，而不能完全剔除，所以才有了常见对的关键字 null。

# 避免使用 null 检查
空指针异常是一个运行时异常，对于这一类异常，如果没有明确的处理方式，那么最佳实践在于让程序早点挂掉。当异常真的发生的时候，处理方式也很简单，在存在异常的地方添加一个 if 语句判定即可。比如下面的代码：

```
public String bindUserToRole(User user) {
    if (user == null) {
        return;
    }

    String roleId = user.getRoleId();
    if (roleId == null) {
        return;
    }

    Role = roleDao.findOne(roleId);
    if (role != null) {
        role.setUserId(user.getUserId());
        roleDao.save(role);
    }
}
```

但是这样的应对方式会让程序出现越来越多的 null 判定，一个良好的程序设计，应该让代码中尽量少出现 null 关键字，因此 Java 8 引入 Optional 类来避免 NPE 问题，同时也提升了代码的美观度。但并不是对 null 关键字的一种替代，而是对于 null 判定提供了一种更加优雅的实现，从而避免 NPE 问题。

# Optional 类
为了更好的解决和避免常见的 NPE 问题，Java 8 中引入了一个新的类 java.util.Optional<T>，Optional 值可以为 null，如果值存在，调用 isPresent() 方法返回 true，调用 get() 方法可以获取值。

## 创建 Optional 对象
Optional 类提供类三个方法用于实例化一个 Optional 对象，它们分别为 empty()、of()、ofNullable()，这三个方法都是静态方法，可以直接调用。

empty() 方法用于创建一个没有值的Optional对象：

```
Optional<String> emptyOpt = Optional.empty();
```

empty() 方法创建的对象没有值，如果对 emptyOpt 变量调用 isPresent() 方法会返回 false，调用 get() 方法抛出 NPE 异常。

of() 方法使用一个非空的值创建Optional对象：

```
String str = "Hello World";
Optional<String> notNullOpt = Optional.of(str);
```

ofNullable() 方法接收一个可以为null的值：

```
Optional<String> nullableOpt = Optional.ofNullable(str);
```

如果 str 的值为 null，得到的 nullableOpt 是一个没有值的 Optional 对象。

## 获取 Optional 对象中的值
如果我们要获取 User 对象中的 roleId 属性值，常见的方式是直接获取：

```
String roleId = null;
if (user != null) {
    roleId = user.getRoleId();
}
```

使用 Optional 中提供的 map() 方法可以更简单地实现：

```
Optional<User> userOpt = Optional.ofNullable(user);
Optional<String> roleIdOpt = userOpt.map(User::getRoleId);
```

使用 orElse()方法获取值

Optional 类还包含其他方法用于获取值，这些方法分别为：
- orElse()：如果有值就返回，否则返回一个给定的值作为默认值
- orElseGet()：与 orElse() 方法作用类似，区别在于生成默认值的方式不同。该方法接受一个 Supplier<? extends T> 函数式接口参数，用于生成默认值
- orElseThrow()：与前面介绍的 get() 方法类似，当值为 null 时调用这两个方法都会抛出 NPE 异常，区别在于该方法可以指定抛出的异常类型

下面来看看这三个方法的具体用法：

```
String str = "Hello World";
Optional<String> strOpt = Optional.of(str);
String orElseResult = strOpt.orElse("Hello BeiJing");
String orElseGet = strOpt.orElseGet(() -> "Hello BeiJing");
String orElseThrow = strOpt.orElseThrow(
        () -> new IllegalArgumentException("Argument 'str' cannot be null or blank."));
```

此外，Optional 类还提供了一个 ifPresent() 方法，该方法接收一个 Consumer<? super T> 函数式接口，一般用于将信息打印到控制台：

```
Optional<String> strOpt = Optional.of("Hello World");
strOpt.ifPresent(System.out::println);
```

使用 filter() 方法过滤

filter() 方法可用于判断 Optional 对象是否满足给定条件，一般用于条件过滤：

```
Optional<String> optional = Optional.of("wupx94@qq.com");
optional = optional.filter(str -> str.contains("wupx"));
```

在上面的代码中，如果 filter() 方法中的 Lambda 表达式成立，filter() 方法会返回当前 Optional 对象值，否则，返回一个值为空的 Optional 对象。

> 关于 Optional 使用建议：
> 
> - 尽量避免在程序中直接调用 Optional 对象的 get() 和 isPresent() 方法
> - 避免使用 Optional 类型声明实体类的属性

## Optional 实践
上面提到创建 Optional 对象有三个方法，empty() 方法比较简单，主要是 of() 和 ofNullable() 方法。当你确定一个对象不可能为 null 的时候，应该使用 of() 方法，否则，尽可能使用 ofNullable() 方法，比如：

```
public static void method(Role role) {
    // 当Optional的值通过常量获得或者通过关键字 new 初始化，可以直接使用 of() 方法
    Optional<String> strOpt = Optional.of("Hello World");
    Optional<User> userOpt = Optional.of(new User());

    // 方法参数中role值不确定是否为null，使用 ofNullable() 方法创建
    Optional<Role> roleOpt = Optional.ofNullable(role);
}
```

orElse() 方法的使用

```
return str != null ? str : "Hello World"
```

上面的代码表示判断字符串 str 是否为空，不为空就返回，否则，返回一个常量。使用 Optional 类可以表示为：

```
return strOpt.orElse("Hello World")
```

简化 if-else

```
User user = ...
if (user != null) {
    String userName = user.getUserName();
    if (userName != null) {
        return userName.toUpperCase();
    } else {
        return null;
    }
} else {
    return null;
}
```

上面的代码可以简化成：

```
User user = ...
Optional<User> userOpt = Optional.ofNullable(user);

return userOpt.map(User::getUserName)
            .map(String::toUpperCase)
            .orElse(null);
```

## 注意事项
Optional 是一个 final 类，未实现任何接口，Optional 不能序列化，不能作为类的字段(field)，所以当我们在利用该类包装定义类的属性的时候，如果我们定义的类有序列化的需求，那么因为 Optional 没有实现 Serializable 接口，这个时候执行序列化操作就会有问题：

```
import java.util.Optional;
import lombok.Data;

@Data
public class User implements Serializable {
    private String name;
    private String gender;
    private Optional<String> phone; // 不能序列化
}
```

可以通过自己实现 getter 方法，使 Lomok 不自动生成，如下：

```
import java.util.Optional;
import lombok.Data

@Data
public class User implements Serializable  {
    private String name;
    private String gender;
    private String phone;
    public Optional<String> getPhone() {
        return Optional.ofNullable(phone);
    }
}
```

# 总结
Java 8 中 Optional 类可以让我们以函数式编程的方式处理 null 值，抛弃了 Java 8 之前需要嵌套大量 if-else 代码块，使代码可读性有了很大的提高，但是应尽量避免使用 Optional 类型声明实体类的属性。