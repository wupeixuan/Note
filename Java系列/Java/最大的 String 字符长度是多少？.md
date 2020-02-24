String 类可以说是在 Java 中使用最频繁的类了，就算是刚刚接触 Java 的初学者也不会陌生，因为对于 Java 程序来说，main  方法就是使用一个 String 类型数组来作为参数的（String[] args）。对于这样一个频繁使用的类，String 字符串可以有多长呢？十万字符？一百万字符？还是无限的呢？

要弄清楚 String 的最大长度，首先应该了解 String 类的内部实现。在 String 类中，是使用一个字符数组来维护字符序列的，其声明如下：

```
private final char value[];
```

这也就是说，String 的最大长度取决于字符数组的最大长度，我们知道，在指定数组长度的时候，可以使用 byte、short、char、int 类型，而不能够使用 long 类型。这也就是说，数组的最大长度就是 int 类型的最大值，即 0x7fffffff，十进制就是 2147483647，同理，这也就是 String 所能容纳的最大字符数量。

而且，我们来看下 java.lang.String#length() 源码：

```
public int length() {
    return value.length;
}
```

可以看出获得 String 对象长度的 length 方法返回值是 int 类型的，而不是 long 类型的，也是因为这个原因。

不过，这个最大值只是在理论上能够达到的值，在我们实际的使用中，一般情况下获得的最大长度比理论值要小。下面我们写一个最简单的程序来看。

```
/**
 * @author wupx
 * @date 2020/01/13
 */
public class StringTest {
    public static void main(String[] args) {
        char[] c = new char[Integer.MAX_VALUE];
    }
}
```

运行这个程序，在通常情况下，都会产生如下的错误：

```
Exception in thread "main" java.lang.OutOfMemoryError: Requested array size exceeds VM limit
	at test.StringTest.main(StringTest.java:9)
```

产生这个错误的原因就是内存溢出，也就是系统无法分配这么大的内存空间所致。计算一下，一个 char 类型占用 2 字节，2147483647 个 char 类型就是 4294967294 字节，这接近于 4GB 大小，想要申请这么一大块连续的内存空间，失败也就不足为奇了。

那么，到底我们所用的计算机能够承受多大的字符数组呢，这跟软件与硬件等诸多因素都有关，我们可以编写程序来获得可申请最大字符数组的近似值。

```
/**
 * @author wupx
 * @date 2020/01/13
 */
public class StringTest {
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            int len = Integer.MAX_VALUE - i;
            try {
                char[] ch = new char[len];
                System.out.println("len: " + len + " OK");
            } catch (Error e) {
                System.out.println("len: " + len + " " + e);
            }
        }
    }
}
```

运行结果如下：

```
len: 2147483647 java.lang.OutOfMemoryError: Requested array size exceeds VM limit
len: 2147483646 java.lang.OutOfMemoryError: Requested array size exceeds VM limit
len: 2147483645 OK
len: 2147483644 OK
len: 2147483643 OK
```

根据运行结果可以看出 String 的最大长度为 Integer.MAX_VALUE - 2 或 2 ^ 31 - 3。

# 总结

在 String 类内部，是使用一个字符数组（char[]）来维护字符序列的。

String 的最大长度也就是字符数组的最大长度，理论上最大长度为 int 类型的最大值，即 2147483647。

在实际中，一般可获取的最大值小于理论最大值，在我的电脑上得出的最大值是 2 ^ 31 - 3，大家可以在自己的电脑上测试下。