在阅读《阿里巴巴Java开发手册》时，发现有一条关于在 foreach 循环里进行元素的 remove/add 操作的规约，具体内容如下：

![阿里巴巴Java开发手册](https://img-blog.csdnimg.cn/20191202223028397.png)

# 错误演示

我们首先在 IDEA 中编写一个在 foreach 循环里进行 remove 操作的代码：

```
import java.util.ArrayList;
import java.util.List;

public class ForEachTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("wupx");
        list.add("love");
        list.add("huxy");
        for (String temp : list) {
            if ("love".equals(temp)) {
                list.remove(temp);
            }
        }
        System.out.println(list);
    }
}
```

此时执行代码，编译正确，执行成功！输出 [wupx, huxy]。

接着我们把 “love” 换成 “wupx” 或是 “huxy” 再来运行下，执行结果如下：

![](https://img-blog.csdnimg.cn/20191202224321610.png)

纳尼，居然报错了，为什么第一次运行没有报错呢？让我们一起来进行探讨吧！

# 追根溯源

为了研究为什么会出现这样的情况，我们可以根据异常堆栈信息，去追踪错误，其中涉及到的部分源码如下：

```
private class Itr implements Iterator<E> {
    int cursor;       // 下一个要返回的元素的索引
    int lastRet = -1; // 返回的最后一个元素的索引（如果没有返回-1）
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size;
    }
    
    public void remove() {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();

        try {
            ArrayList.this.remove(lastRet);
            cursor = lastRet;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public void forEachRemaining(Consumer<? super E> consumer) {
        Objects.requireNonNull(consumer);
        final int size = ArrayList.this.size;
        int i = cursor;
        if (i >= size) {
            return;
        }
        final Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length) {
            throw new ConcurrentModificationException();
        }
        while (i != size && modCount == expectedModCount) {
            consumer.accept((E) elementData[i++]);
        }
        cursor = i;
        lastRet = i - 1;
        checkForComodification();
    }
    
    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

从代码中可以看出，其实在集合遍历时维护一个初始值为 0 的游标 cursor，从头到尾地进行扫描，在 cursor==size 时，退出遍历。如下图所示，执行 remove 这个元素后，所有元素往前拷贝， size=size-1 即为2 ，这时 cursor 也等于 2。在执行
hasNext() 时， 结果为 false ，退出循环体，并没有机会执行到 next() 的第一行代码
checkForComodification() ，此方法用来判断 expectedModCount 和 modCount 是否相等，
如果不相等，则抛出 ConcurrentModificationException 异常。

![集合的cursor与size](https://img-blog.csdnimg.cn/20191202233208551.png)

之所以会报 ConcurrentModificationException 异常，是因为触发了 Java 的 fail-fast 机制，该机制是集合中比较常见的错误检测机制，通常出现在遍历集合元素的过程中。举个生活中的栗子：

比如上体育课时，在上课前都会依次报数，如果在报数期间，有人突然加进来，还要重新报数，再次报数，又有同学溜出去了，又要重新报数，这就是 fail-fast 机制，它是对集合（班级同学）遍历操作的错误检测机制，在遍历中途出现意料之外的修改时，通过 unchecked 异常反馈出来。这种机制经常出现在多线程环境下，当前线程会维护一个计数比较器（expectedModCount），记录已经修改的次数。在进入遍历前，会把实时修改次数
 modCount 赋值给 expectedModCount，如果这两个数据不相等，则抛出异常。java.util 下的所有集合类都是 fail-fast。

# 不二法门
既然在 foreach 循环里进行元素的 remove/add 操作会有问题，那么我们可以使用手册中推荐的 Iterator 机制进行遍历时的删除或新增，代码如下：

```
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class ForEachTest {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("wupx");
        list.add("love");
        list.add("huxy");

        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            if (iterator.next().equals("wupx")) {
                iterator.remove();
            }
        }
        System.out.println(list);
    }
}
```

如果是多线程并发，还需要在 Iterator 遍历时加锁，或者使用并发容器 CopyOnWriteArrayList 代替 ArrayList，该容器内部会对 Iterator 进行加锁操作。

# 总结
本文针对《阿里巴巴Java开发手册》中的强制要求不要在 foreach 循环里进行元素的 remove/add 操作出发，从源码层面来解释为什么，还用生活中的栗子来介绍 Java 中的 fail-fast 机制，因此在进行元素的 remove/add 操作时要用 Iterator 去遍历删除或新增。

> 参考
> 
> 《Java开发手册》华山版
>
> 《码出高效：Java开发手册》