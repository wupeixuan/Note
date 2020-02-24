这篇文章主要来介绍下 JVM 中的各种 GC，让大家来搞清楚这几个概念。

大家可能见到过很多的 GC 名词，比如：Minor GC、Young GC、Full GC、Old GC、Major GC、Mixed GC。

这么多概念，想想都头疼，到底各种乱七八糟的 GC 指的是什么？

下面先引用 R 大在知乎上的回答：

针对 HotSpot VM 的实现，它里面的 GC 其实准确分类有两种:

- Partial GC(局部 GC): 并不收集整个 GC 堆的模式
    - Young GC: 只收集 Young Gen 的 GC，Young GC 还有种说法就叫做 Minor GC
    - Old GC: 只收集 old gen 的 GC，只有垃圾收集器 CMS 的 concurrent collection 是这个模式
    - Mixed GC: 收集整个 Young Gen 以及部分 old gen 的 GC，只有垃圾收集器 G1 有这个模式
- Full GC: 收集整个堆，包括新生代，老年代，永久代(在 JDK 1.8 及以后，永久代被移除，换为 metaspace 元空间)等所有部分的模式

接下来让我们再来了解下各个 GC：

（1）Minor GC / Young GC

首先我们先来看下 Minor GC / Young GC，大家都知道，新生代（Young Gen）也可以称之为年轻代，这两个名词是等价的。那么在年轻代中的 Eden 内存区域被占满之后，实际上就需要触发年轻代的 GC，或者是新生代的 GC。

此时这个新生代 GC，其实就是所谓的 Minor GC，也可以称之为 Young GC，这两个名词，相信大家就理解了，说白了，就专门针对新生代的 GC。

（2）Old GC

所谓的老年代 GC，称之为 Old GC 更加合适一些，因为从字面意义上就可以理解，这就是所谓的老年代 GC。

但是在这里之所以我们把老年代 GC 称之为Full GC，其实也是可以的，只不过是一个字面意思的多种不同的说法。

为了更加精准的表述这个老年代 GC 的含义，可以把老年代 GC 称之为 Old GC。

（3）Full GC

对于 Full GC，其实这里有一个更加合适的说法，就是说 Full GC 指的是针对新生代、老年代、永久代的全体内存空间的垃圾回收，所以称之为 Full GC。

从字面意思上也可以理解，Full 就是整体的意思，所以就是对 JVM 进行一次整体的垃圾回收，把各个内存区域的垃圾都回收掉。

（4）Major GC

还有一个名词是所谓的 Major GC，这个其实一般用的比较少，他也是一个非常容易混淆的概念。

有些人把 Major GC 跟 Old GC等价起来，认为他就是针对老年代的 GC，也有人把 Major GC 和 Full GC 等价起来，认为他是针对 JVM 全体内存区域的GC。

所以针对这个容易混淆的概念，建议大家以后少提。如果听到有人说这个 Major GC的概念，大家可以问清楚，他到底是想说 Old GC 呢？还是 Full GC 呢？

（5）Mixed GC

Mixed GC 是 G1 中特有的概念，其实说白了，主要就是说在 G1 中，一旦老年代占据堆内存的 45%（-XX:InitiatingHeapOccupancyPercent：设置触发标记周期的 Java 堆占用率阈值，默认值是 45%。这里的Java 堆占比指的是 non_young_capacity_bytes，包括 old + humongous），就要触发 Mixed GC，此时对年轻代和老年代都会进行回收。Mixed GC 只有 G1 中才会出现。

> 参考
> 
> https://tech.meituan.com/2016/09/23/g1.html
> 
> https://www.zhihu.com/question/41922036/answer/93079526
> 
> 《深入理解Java虚拟机：JVM高级特性与最佳实践（第3版）》