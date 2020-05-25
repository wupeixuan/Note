在阅读《阿里巴巴Java开发手册》时，发现有一条关于关于常量定义的规约，具体内容如下：

![](https://img-blog.csdnimg.cn/20200524165401479.png)

图中的反例是将数据缓存起来，并使用魔法值加链路 id 组成 key，这就可能会出现其他开发人员在复制粘贴的时候，少复制 `_` 的情况发生，这种错误很难去检查到，因为读取缓存不存在，可能会去数据库读取，很难察觉到。

如果在生产环境中，大量的请求进来，缓存全部失效，直接请求数据库，导致数据库连接过多，查询效率变低的问题发生，因此看来魔法值确实应该避免出现在代码中。

另外在 《Clean Code》 和 《重构》 等书中也提到了类似的问题，在代码中出现原始形态数字通常来说是坏现象，应该用命名良好的常量类隐藏它。

## 静态常量取代魔法值

像下面这个例子：

```
if (billCount > 75) {
    //todo
} else {
    //todo
}
```

如果在不了解这块的业务的同事，在读到这块代码的时候，可能会想，`75` 是什么鬼，为啥和这个数比较，背后深藏着什么秘密吗？可能只有当时的开发人员记得了，导致代码可读性和可维护性极差。

如果声明一个常量，来替换该魔法值，可能就会使代码的可读性和可维护性大大增加。

```
static final Integer BASIC_BILL_COUNT = 75;
```

还有些魔法表达式，比如：

```
if (value > 60 && value <= 80 && type = 1) {
    // todo
}
```

比如这个表达式是表示状态为正常且项目活跃，就可以定义：

```
boolean isActiveProject = value > 60 && value <= 80 && type = 1;
```

这样是不是可读性就提高了，一眼就可以看出来这块代码的逻辑。

## 枚举类取代魔法值

还有一种消除魔法值的方式是使用枚举类代替，下面让我们举个例子：

```
if (eventId == 1) {
    System.out.println("睡觉");
} else if (eventId == 2) {
    System.out.println("吃饭");
} else if (eventId == 3) {
    System.out.println("打豆豆");
}
```

如上代码是针对事件 id 去执行相应的事件，如果事件比较少，大家还可以勉强记住每个 eventId 对应的含义，但是随着事件 id 的增多，很可能会发生，新来的员工把事件 id 给搞混了，导致执行错误的事件，发生 bug。

那么我们可以使用枚举类来表示相应的事件：

```
public enum EventEnum {

    /**
     * 睡觉
     */
    SLEEP_EVENT(1, "睡觉"),

    /**
     * 吃饭
     */
    EAT_EVENT(2, "吃饭"),

    /**
     * 打豆豆
     */
    FIGHT_PEA_EVENT(3, "打豆豆");

    private int eventId;
    private String desc;

    EventEnum(int eventId, String desc) {
        this.eventId = eventId;
        this.desc = desc;
    }

    public int getEventId() {
        return eventId;
    }

    public String getDesc() {
        return desc;
    }
}
```

修改完之后的代码如下：

```
if (eventId == EventEnum.SLEEP_EVENT.getEventId()) {
    System.out.println("睡觉");
} else if (eventId == EventEnum.EAT_EVENT.getEventId()) {
    System.out.println("吃饭");
} else if (eventId == EventEnum.FIGHT_PEA_EVENT.getEventId()) {
    System.out.println("打豆豆");
}
```

是不是可读性急剧提升，还不快看看自己代码中有没有这样的魔法值出现，有的话赶紧改造起来。

还有如果你需要在不同的地点引用同一数值，魔法数会让你烦恼不已，因为一旦这些数字发生改变，就必须在程序中找到所有的魔法值，并将它们全部修改一遍，这样就太费时费力了。

其实不只是 Java 不应该在代码中使用魔法值，其他语言亦是如此。

# 总结

本文主要介绍了为什么不允许在代码中出现魔法值以及如何将代码中已有的魔法值去除掉。

代码可读性还是比较重要的，你肯定不希望别人在接手你的代码的时候，骂到这数字啥意思，这代码写得跟粑粑一样。

**最好的关系就是互相成就**，大家的**在看、转发、留言**三连就是我创作的最大动力。

> 参考
> 
> 《Java开发手册》泰山版