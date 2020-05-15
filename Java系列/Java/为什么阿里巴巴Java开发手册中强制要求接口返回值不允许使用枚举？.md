在阅读《阿里巴巴Java开发手册》时，发现有一条关于二方库依赖中接口返回值不允许使用枚举类型的规约，具体内容如下：

![](https://img-blog.csdnimg.cn/20200514234639139.png)

在谈论为什么之前先来科普下什么是二方库，**二方库**也称作二方包，一般指公司内部发布到中央仓库，可供公司内部其他应用依赖的库（jar 包）。

那么**一方库**便是本工程内部子项目模块依赖的库；**三方库**为公司之外的开源库，比如像 `fastjson、easyexcel` 这种。

下面我们就通过一个例子来看下为什么阿里巴巴不允许返回枚举类型或者包含枚举类型的 POJO 对象。

比如星巴克提供了 `0.0.1` 版本的二方库，定义了一个 `Starbucks` 类，里面包含了枚举类型的 `SizeEnum`，里面分别是中杯、大杯、特大杯。

```
public class Starbucks implements Serializable {
  private Long id;
  private String name;
  private Integer capacity;
  private SizeEnum sizeEum;
}

public enum SizeEnum {
  TALL(1),
  GRANDE(2),
  VENTI(3)
}
```

定义了一个服务类，实现了根据 `id` 获取星巴克的方法：

```
public class StarbucksImpl implements StarbucksService {
  public Starbucks getStarbucksById(Long id) {
    Starbucks starbucks = new Starbucks();
    starbucks.setId(1L);
    starbucks.setName("Latte");
    starbucks.setCapacity(360);
    starbucks.setSizeEnum(SizeEnum.TALL);
    return starbucks;
  }
}
```

然后，星巴克的门店引入 `0.0.1` 这个版本 jar 包，然后卖的好好的：

```
public class StarbucksDemo {
  @Resource
  private StarbucksService starbucksService;
  
  public void getStarbucks() {
    Starbucks starbucks = starbucksService.getStarbucksById(1L);
    System.out.println(starbucks);
  }
}
```

有一天，老罗说要那个中等大小的中杯拿铁，但是服务员说那是大杯，经过一番争论，罗老师很是生气。

![](https://img-blog.csdnimg.cn/20200515010521218.gif)

于是星巴克升级到了 `0.0.2` 版本二方库，在枚举类 `SizeEnum` 中新增了小杯，升级后的枚举类如下：

```
public enum SizeEnum {
  TALL(1),
  GRANDE(2),
  VENTI(3),
  SHORT(4)
}
```

同时服务类的接口方法也做了相应修改：

```
public class StarbucksImpl implements StarbucksService {
  public Starbucks getStarbucksById(Long id) {
    Starbucks starbucks = new Starbucks();
    starbucks.setId(1L);
    starbucks.setName("Latte");
    starbucks.setCapacity(240);
    starbucks.setSizeEnum(SizeEnum.SHORT);
    return starbucks;
  }
}
```

由于星巴克的门店比较多，有的还不知道这个新加的需求，因此返回结果中出现了 `SHORT`，但是 `0.0.1` 版本的二方库中没有小杯啊，所以就出问题了，也就是序列化失败。

通过这个例子，我相信大家对枚举类型作为返回结果有了一定的理解，下面引用孤尽大佬在知乎的回答：

> 由于升级原因，导致双方的枚举类不尽相同，在接口解析，类反序列化时出现异常。
>
> Java 中出现的任何元素，在 Gosling 的角度都会有背后的思考和逻辑（尽管并非绝对完美，但 Java 的顶层抽象已经是天才级了），比如：接口、抽象类、注解、和本文提到的枚举。枚举有好处，类型安全，清晰直接，还可以使用等号来判断，也可以用在 switch 中。它的劣势也是明显的，就是不要扩展。可是为什么在返回值和参数进行了区分呢，如果不兼容，那么两个都有问题，怎么允许参数可以有枚举。当时的考虑，如果参数也不能用，那么枚举几乎无用武之地了。参数输出，毕竟是本地决定的，你本地有的，传送过去，向前兼容是不会有问题的。但如果是接口返回，就比较恶心了，因为解析回来的这个枚举值，可能本地还没有，这时就会抛出序列化异常。
>
> 比如：你的本地枚举类，有一个天气 Enum：SUNNY, RAINY, CLOUDY，如果根据天气计算心情的方法：guess(WeatcherEnum xx)，传入这三个值都是可以的。返回值：Weather guess(参数)，那么对方运算后，返回一个 SNOWY，本地枚举里没有这个值，傻眼了。

# 总结

本文通过一个实例让大家理解到枚举类型作为返回结果的坑，大家可以使用基本类型或者基本类型包装类来替换掉枚举类型就可以避免掉这么问题了。

大家对于这条规约​有什么看法，也欢迎留言讨论。​

**最好的关系就是互相成就**，大家的**在看、转发、留言**三连就是我创作的最大动力。

> 参考
> 
> 《Java开发手册》泰山版
>
> https://www.zhihu.com/question/52760637/answer/338584321