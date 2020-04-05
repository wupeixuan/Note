当多线程访问共享可变数据时，涉及到线程间同步的问题，并不是所有时候，都要用到共享数据，所以就需要线程封闭出场了。

数据都被封闭在各自的线程之中，就不需要同步，这种通过将数据封闭在线程中而避免使用同步的技术称为**线程封闭**。

本文主要介绍线程封闭中的其中一种体现：ThreadLocal，将会介绍什么是 ThreadLocal；从 ThreadLocal 源码角度分析，最后介绍 ThreadLocal 的应用场景。

## 什么是 ThreadLocal？

ThreadLocal 是 Java 里一种特殊变量，它是一个线程级别变量，每个线程都有一个 ThreadLocal 就是每个线程都拥有了自己独立的一个变量，竞态条件被彻底消除了，在并发模式下是绝对安全的变量。

可以通过 `ThreadLocal<T> value = new ThreadLocal<T>();` 来使用。

会自动在每一个线程上创建一个 T 的副本，副本之间彼此独立，互不影响，可以用 ThreadLocal 存储一些参数，以便在线程中多个方法中使用，用以代替方法传参的做法。

下面通过例子来了解下 ThreadLocal：

```
public class ThreadLocalDemo {
    /**
     * ThreadLocal变量，每个线程都有一个副本，互不干扰
     */
    public static final ThreadLocal<String> THREAD_LOCAL = new ThreadLocal<>();

    public static void main(String[] args) throws Exception {
        new ThreadLocalDemo().threadLocalTest();
    }

    public void threadLocalTest() throws Exception {
        // 主线程设置值
        THREAD_LOCAL.set("wupx");
        String v = THREAD_LOCAL.get();
        System.out.println("Thread-0线程执行之前，" + Thread.currentThread().getName() + "线程取到的值：" + v);

        new Thread(new Runnable() {
            @Override
            public void run() {
                String v = THREAD_LOCAL.get();
                System.out.println(Thread.currentThread().getName() + "线程取到的值：" + v);
                // 设置 threadLocal
                THREAD_LOCAL.set("huxy");
                v = THREAD_LOCAL.get();
                System.out.println("重新设置之后，" + Thread.currentThread().getName() + "线程取到的值为：" + v);
                System.out.println(Thread.currentThread().getName() + "线程执行结束");
            }
        }).start();
        // 等待所有线程执行结束
        Thread.sleep(3000L);
        v = THREAD_LOCAL.get();
        System.out.println("Thread-0线程执行之后，" + Thread.currentThread().getName() + "线程取到的值：" + v);
    }
}
```

首先通过 `static final` 定义了一个 `THREAD_LOCAL` 变量，其中 `static` 是为了确保全局只有一个保存 String 对象的 ThreadLocal 实例；`final` 确保 ThreadLocal 的实例不可更改，防止被意外改变，导致放入的值和取出来的不一致，另外还能防止 ThreadLocal 的内存泄漏。上面的例子是演示在不同的线程中获取它会得到不同的结果，运行结果如下：

```
Thread-0线程执行之前，main线程取到的值：wupx
Thread-0线程取到的值：null
重新设置之后Thread-0线程取到的值为：huxy
Thread-0线程执行结束
Thread-0线程执行之后，main线程取到的值：wupx
```

首先在 `Thread-0` 线程执行之前，先给 `THREAD_LOCAL` 设置为 `wupx`，然后可以取到这个值，然后通过创建一个新的线程以后去取这个值，发现新线程取到的为 null，意外着这个变量在不同线程中取到的值是不同的，不同线程之间对于 ThreadLocal 会有对应的副本，接着在线程 `Thread-0` 中执行对 `THREAD_LOCAL` 的修改，将值改为 `huxy`，可以发现线程 `Thread-0` 获取的值变为了 `huxy`，主线程依然会读取到属于它的副本数据 `wupx`，这就是线程的封闭。

看到这里，我相信大家一定会好奇 ThreadLocal 是如何做到多个线程对同一对象 set 操作，但是 get 获取的值还都是每个线程 set 的值呢，接下来就让我们进入源码解析环节：

## ThreadLocal 源码解析

首先看下 ThreadLocal 都有哪些重要属性：

```
// 当前 ThreadLocal 的 hashCode，由 nextHashCode() 计算而来，用于计算当前 ThreadLocal 在 ThreadLocalMap 中的索引位置
private final int threadLocalHashCode = nextHashCode();
// 哈希魔数，主要与斐波那契散列法以及黄金分割有关
private static final int HASH_INCREMENT = 0x61c88647;
// 返回计算出的下一个哈希值，其值为 i * HASH_INCREMENT，其中 i 代表调用次数
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
// 保证了在一台机器中每个 ThreadLocal 的 threadLocalHashCode 是唯一的
private static AtomicInteger nextHashCode = new AtomicInteger();
```

其中的 `HASH_INCREMENT` 也不是随便取的，它转化为十进制是 `1640531527`，`2654435769` 转换成 int 类型就是 `-1640531527`，`2654435769` 等于 `(√5-1)/2` 乘以 2 的 32 次方。`(√5-1)/2` 就是黄金分割数，近似为 `0.618`，也就是说 `0x61c88647` 理解为一个黄金分割数乘以 2 的 32 次方，它可以保证 nextHashCode 生成的哈希值，均匀的分布在 2 的幂次方上，且小于 2 的 32 次方。

下面是 javaspecialists 中一篇文章对它的介绍：

> This number represents the golden ratio (sqrt(5)-1) times two to the power of 31 ((sqrt(5)-1) * (2^31)). The result is then a golden number, either 2654435769 or -1640531527.

下面用例子来证明下：

```
private static final int HASH_INCREMENT = 0x61c88647;

public static void main(String[] args) throws Exception {
    int n = 5;
    int max = 2 << (n - 1);
    for (int i = 0; i < max; i++) {
        System.out.print(i * HASH_INCREMENT & (max - 1));
        System.out.print(" ");

    }
}
```

运行结果为：`0 7 14 21 28 3 10 17 24 31 6 13 20 27 2 9 16 23 30 5 12 19 26 1 8 15 22 29 4 11 18 25 `

可以发现元素索引值完美的散列在数组当中，并没有出现冲突。

### ThreadLocalMap

除了上述属性外，还有一个重要的属性 ThreadLocalMap，ThreadLocalMap 是 ThreadLocal 的静态内部类，当一个线程有多个 ThreadLocal 时，需要一个容器来管理多个 ThreadLocal，ThreadLocalMap 的作用就是管理线程中多个 ThreadLocal，源码如下：

```
static class ThreadLocalMap {
	/**
	 * 键值对实体的存储结构
	 */
	static class Entry extends WeakReference<ThreadLocal<?>> {
		// 当前线程关联的 value，这个 value 并没有用弱引用追踪
		Object value;

		/**
		 * 构造键值对
		 *
		 * @param k k 作 key,作为 key 的 ThreadLocal 会被包装为一个弱引用
		 * @param v v 作 value
		 */
		Entry(ThreadLocal<?> k, Object v) {
			super(k);
			value = v;
		}
	}

	// 初始容量，必须为 2 的幂
	private static final int INITIAL_CAPACITY = 16;

	// 存储 ThreadLocal 的键值对实体数组，长度必须为 2 的幂
	private Entry[] table;

	// ThreadLocalMap 元素数量
	private int size = 0;

	// 扩容的阈值，默认是数组大小的三分之二
	private int threshold;
}
```

从源码中看到 ThreadLocalMap 其实就是一个简单的 Map 结构，底层是数组，有初始化大小，也有扩容阈值大小，数组的元素是 Entry，**Entry 的 key 就是 ThreadLocal 的引用，value 是 ThreadLocal 的值**。ThreadLocalMap 解决 hash 冲突的方式采用的是**线性探测法**，如果发生冲突会继续寻找下一个空的位置。

这样的就有可能会发生内存泄漏的问题，下面让我们进行分析：

### ThreadLocal 内存泄漏

ThreadLocal 在没有外部强引用时，发生 GC 时会被回收，那么 ThreadLocalMap 中保存的 key 值就变成了 null，而 Entry 又被 threadLocalMap 对象引用，threadLocalMap 对象又被 Thread 对象所引用，那么当 Thread 一直不终结的话，value 对象就会一直存在于内存中，也就导致了内存泄漏，直至 Thread 被销毁后，才会被回收。

那么如何避免内存泄漏呢？

在使用完 ThreadLocal 变量后，需要我们手动 remove 掉，防止 ThreadLocalMap 中 Entry 一直保持对 value 的强引用，导致 value 不能被回收，其中 remove 源码如下所示：

```
/**
 * 清理当前 ThreadLocal 对象关联的键值对
 */
public void remove() {
	// 返回当前线程持有的 map
	ThreadLocalMap m = getMap(Thread.currentThread());
	if (m != null) {
		// 从 map 中清理当前 ThreadLocal 对象关联的键值对
		m.remove(this);
	}
}
```

remove 方法的时序图如下所示：

![](https://img-blog.csdnimg.cn/20200405150646681.png)

remove 方法是先获取到当前线程的 ThreadLocalMap，并且调用了它的 remove 方法，从 map 中清理当前 ThreadLocal 对象关联的键值对，这样 value 就可以被 GC 回收了。

那么 ThreadLocal 是如何实现线程隔离的呢？

### ThreadLocal 的 set 方法

我们先去看下 ThreadLocal 的 set 方法，源码如下：

```
/**
 * 为当前 ThreadLocal 对象关联 value 值
 *
 * @param value 要存储在此线程的线程副本的值
 */
public void set(T value) {
	// 返回当前ThreadLocal所在的线程
	Thread t = Thread.currentThread();
	// 返回当前线程持有的map
	ThreadLocalMap map = getMap(t);
	if (map != null) {
		// 如果 ThreadLocalMap 不为空，则直接存储<ThreadLocal, T>键值对
		map.set(this, value);
	} else {
		// 否则，需要为当前线程初始化 ThreadLocalMap，并存储键值对 <this, firstValue>
		createMap(t, value);
	}
}
```

set 方法的作用是把我们想要存储的 value 给保存进去。set 方法的流程主要是：

- 先获取到当前线程的引用
- 利用这个引用来获取到 ThreadLocalMap
- 如果 map 为空，则去创建一个 ThreadLocalMap
- 如果 map 不为空，就利用 ThreadLocalMap 的 set 方法将 value 添加到 map 中

set 方法的时序图如下所示：

![](https://img-blog.csdnimg.cn/2020040514180914.png)

其中 map 就是我们上面讲到的 ThreadLocalMap，可以看到它是通过当前线程对象获取到的 ThreadLocalMap，接下来我们看 getMap方法的源代码：

```
/**
 * 返回当前线程 thread 持有的 ThreadLocalMap
 *
 * @param t 当前线程
 * @return ThreadLocalMap
 */
ThreadLocalMap getMap(Thread t) {
	return t.threadLocals;
}
```

getMap 方法的作用主要是获取当前线程内的 ThreadLocalMap 对象，原来这个 ThreadLocalMap 是线程的一个属性，下面让我们看看 Thread 中的相关代码：

```
/**
 * ThreadLocal 的 ThreadLocalMap 是线程的一个属性，所以在多线程环境下 threadLocals 是线程安全的
 */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

可以看出每个线程都有 ThreadLocalMap 对象，被命名为 `threadLocals`，默认为 null，所以每个线程的 ThreadLocals 都是隔离独享的。

调用 ThreadLocalMap.set() 时，会把当前 `threadLocal` 对象作为 key，想要保存的对象作为 value，存入 map。

其中 ThreadLocalMap.set() 的源码如下：

```
/**
 * 在 map 中存储键值对<key, value>
 *
 * @param key   threadLocal
 * @param value 要设置的 value 值
 */
private void set(ThreadLocal<?> key, Object value) {
	Entry[] tab = table;
	int len = tab.length;
	// 计算 key 在数组中的下标
	int i = key.threadLocalHashCode & (len - 1);
	// 遍历一段连续的元素，以查找匹配的 ThreadLocal 对象
	for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
		// 获取该哈希值处的ThreadLocal对象
		ThreadLocal<?> k = e.get();

		// 键值ThreadLocal匹配，直接更改map中的value
		if (k == key) {
			e.value = value;
			return;
		}

		// 若 key 是 null，说明 ThreadLocal 被清理了，直接替换掉
		if (k == null) {
			replaceStaleEntry(key, value, i);
			return;
		}
	}

	// 直到遇见了空槽也没找到匹配的ThreadLocal对象，那么在此空槽处安排ThreadLocal对象和缓存的value
	tab[i] = new Entry(key, value);
	int sz = ++size;
	// 如果没有元素被清理，那么就要检查当前元素数量是否超过了容量阙值(数组大小的三分之二)，以便决定是否扩容
	if (!cleanSomeSlots(i, sz) && sz >= threshold) {
		// 扩容的过程也是对所有的 key 重新哈希的过程
		rehash();
	}
}
```

相信到这里，大家应该对 Thread、ThreadLocal 以及 ThreadLocalMap 的关系有了进一步的理解，下图为三者之间的关系：

![](https://img-blog.csdnimg.cn/20200405125741464.png)

### ThreadLocal 的 get 方法

了解完 set 方法后，让我们看下 get 方法，源码如下：

```
/**
 * 返回当前 ThreadLocal 对象关联的值
 *
 * @return
 */
public T get() {
	// 返回当前 ThreadLocal 所在的线程
	Thread t = Thread.currentThread();
	// 从线程中拿到 ThreadLocalMap
	ThreadLocalMap map = getMap(t);
	if (map != null) {
		// 从 map 中拿到 entry
		ThreadLocalMap.Entry e = map.getEntry(this);
		// 如果不为空，读取当前 ThreadLocal 中保存的值
		if (e != null) {
			@SuppressWarnings("unchecked")
			T result = (T) e.value;
			return result;
		}
	}
	// 若 map 为空，则对当前线程的 ThreadLocal 进行初始化，最后返回当前的 ThreadLocal 对象关联的初值，即 value
	return setInitialValue();
}
```

get 方法的主要流程为：

- 先获取到当前线程的引用
- 获取当前线程内部的 ThreadLocalMap
- 如果 map 存在，则获取当前 ThreadLocal 对应的 value 值
- 如果 map 不存在或者找不到 value 值，则调用 setInitialValue() 进行初始化

get 方法的时序图如下所示：

![](https://img-blog.csdnimg.cn/20200405141521235.png)

其中每个 Thread 的 ThreadLocalMap 以 `threadLocal` 作为 key，保存自己线程的 `value` 副本，也就是保存在每个线程中，并没有保存在 ThreadLocal 对象中。

其中 ThreadLocalMap.getEntry() 方法的源码如下：

```
/**
 * 返回 key 关联的键值对实体
 *
 * @param key threadLocal
 * @return
 */
private Entry getEntry(ThreadLocal<?> key) {
	int i = key.threadLocalHashCode & (table.length - 1);
	Entry e = table[i];
	// 若 e 不为空，并且 e 的 ThreadLocal 的内存地址和 key 相同，直接返回
	if (e != null && e.get() == key) {
		return e;
	} else {
		// 从 i 开始向后遍历找到键值对实体
		return getEntryAfterMiss(key, i, e);
	}
}
```

### ThreadLocalMap 的 resize 方法

当 ThreadLocalMap 中的 ThreadLocal 的个数超过容量阈值时，ThreadLocalMap 就要开始扩容了，我们一起来看下 resize 的源代码：

```
/**
 * 扩容，重新计算索引，标记垃圾值，方便 GC 回收
 */
private void resize() {
	Entry[] oldTab = table;
	int oldLen = oldTab.length;
	int newLen = oldLen * 2;
	// 新建一个数组，按照2倍长度扩容
	Entry[] newTab = new Entry[newLen];
	int count = 0;

	// 将旧数组的值拷贝到新数组上
	for (int j = 0; j < oldLen; ++j) {
		Entry e = oldTab[j];
		if (e != null) {
			ThreadLocal<?> k = e.get();
			// 若有垃圾值，则标记清理该元素的引用，以便GC回收
			if (k == null) {
				e.value = null;
			} else {
				// 计算 ThreadLocal 在新数组中的位置
				int h = k.threadLocalHashCode & (newLen - 1);
				// 如果发生冲突，使用线性探测往后寻找合适的位置
				while (newTab[h] != null) {
					h = nextIndex(h, newLen);
				}
				newTab[h] = e;
				count++;
			}
		}
	}
	// 设置新的扩容阈值，为数组长度的三分之二
	setThreshold(newLen);
	size = count;
	table = newTab;
}
```

resize 方法主要是进行扩容，同时会将垃圾值标记方便 GC 回收，扩容后数组大小是原来数组的两倍。

## ThreadLocal 应用场景

ThreadLocal 的特性也导致了应用场景比较广泛，主要的应用场景如下：

- 线程间数据隔离，各线程的 ThreadLocal 互不影响
- 方便同一个线程使用某一对象，避免不必要的参数传递
- 全链路追踪中的 traceId 或者流程引擎中上下文的传递一般采用 ThreadLocal
- Spring 事务管理器采用了 ThreadLocal
- Spring MVC 的 RequestContextHolder 的实现使用了 ThreadLocal

# 总结

本文主要从源码的角度解析了 ThreadLocal，并分析了发生内存泄漏的原因，最后对它的应用场景进行了简单介绍。

欢迎留言交流讨论，原创不易，觉得文章不错，请在看转发支持一下。

更详细的源码解析可以点击链接查看：https://github.com/wupeixuan/JDKSourceCode1.8

> 参考
>
> 《Java并发编程实战》
>
> https://www.javaspecialists.eu/archive/Issue164.html
>
> https://mp.weixin.qq.com/s/vURwBPgVuv4yGT1PeEHxZQ
>
> Java并发编程学习宝典
>
> 面试官系统精讲Java源码及大厂真题
>
> Java 并发面试 78 讲