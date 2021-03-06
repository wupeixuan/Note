# 写在前面
打开 DDD 相关的书籍，你会被一系列生硬、高深的概念充斥，拜读完毕，满头雾水。这不是你的问题，而是 DDD 本身的问题，表现形式太概念化。学习它的内核，就不要被它给出的概念所迷惑，而要去思索这些概念背后所蕴含的设计原则，多问一些为什么，本质无外乎是 SOLID。最重要的，要学会运用设计原则去解决问题，而非所谓的 “设计规范”。

本文将会以系列解答的方式展开，由浅入深，篇幅不长，无妨一看。

# 啥是 DDD？
本质上是一种方法论，提供了一套系统开发的设计方法。面对需要解决的问题，从复杂的现实中抽象出业务模型的思维方式与实践技巧。初衷是清晰设计思路，规范设计过程。

# 啥是驱动？
DDD 强调是说得先把 “领域” 中涉及到的数据、流程、规则等都弄明白了，然后以面向对象的观点为其建立一个模型（即领域模型），而这个模型，决定了你将用什么技术、什么架构、什么平台来实现这个系统。所以技术在这个过程中是 “被动的”，是被 “选来” 实现 “领域模型” 的。对于项目的成败，技术不是决定性因素，领域模型是否符合事物的本质才是关键。

可以看出，领域驱动设计的出发点是业务导向，技术服务于业务。

# 我有误解？
学习 DDD 有一些常见的误区。第一个要避免的就是，你必须要清楚，DDD 对于工程领域没有提出多么创新的技法，它更多是把前人在生产系统中用惯的技法归纳，总结包装了一下：

1. DDD 抛开它晦涩的表达和术语，内核无外乎是 SOLID。书中的技法不是作者的发明创造或独门秘籍，而是当时业已存在并已在使用的做法，只不过没有被系统地加以记录 —— 换而言之，只要遵循 SOLID 的原则，这些所谓的概念技法完全可能在无意识的状态下自发出现在生产代码中。
2. 这些技法在各种大型系统被多次使用。换而言之，只要你接触足够多的代码、足够多的项目，必然会接触到这些 DDD 的实际应用。只不过在看过 DDD 一书之前，你可能意识不到这是一个成体系的设计手段。—— 如果你无法理解这样一个设计理论，看实际生产代码的应用场景是唯一真正有效的方法，毕竟设计理论本身就是从具体代码中总结出来的。即使你觉得自己懂了，抽象思维和开发经验也可能还未达到正确使用它的水平。
3. DDD 最大的价值在于对设计手段进行了有效的整理，形成了一个完整的系统设计规范，但过于晦涩和概念化的表述，也几乎消解了它对业界的贡献。

# 啥时候用？
你可能认为 DDD 是一把 “瑞士军刀”，能够解决所有的设计问题，而实际上 “DDD 只是一把锤子”，有个谚语叫做 “如果你手里有一把锤子，那么所有的问题都变成了钉子”，如果你拿着 DDD 这把锤子到处去敲，要么东西被敲坏，要么就不起作用。

为什么说 DDD 只是一把锤子呢？作者明确指出，DDD 只适合业务复杂度很大的场景，不适用于技术复杂性很大但业务领域复杂性很低的场景。可以看出，DDD 只专注一个领域，高复杂业务 —— 通过它可以为你的系统建立一个核心、稳定的领域模型，灵活、可扩展的架构。

DDD 是拥抱复杂的，拥抱变化的，但本身也是有成本有前提的；简单系统，没必要搞这么复杂，强上 DDD 就是一种反模式了。所以，当你遇到一个问题就想到 DDD 的时候，一定要注意 “DDD 只是一把锤子”，不要拿着这把锤子到处去敲！

# 啥是复杂？
如何判断业务是否复杂，判断依据不胜繁数。在我看来，没那么复杂，就两个：

- 宽度：链路广度大，关注多个纬度的消息来源，覆盖了较多的业务场景
- 深度：链路深度深，关注对象整个的生命周期，从数据创建、到变更、再到后期运维，流程运转长

只要满足其中一个，我认为就是复杂的。

# 具体解决啥？
DDD 并非 “银弹”，自然也不是解决所有疑难杂症的 “灵丹妙药”。在我看来，它只解决一个问题：过度耦合。

# 为啥会耦合？
业务初期，系统功能大都非常简单，CRUD 就能满足，此时的系统是清晰的。随着业务的不断演化，系统的频繁迭代，代码逻辑变得越来越复杂，系统也越来越冗余。模块彼此关联，谁都很难说清模块的具体功能意图是啥；修改一个功能，往往光回溯该功能需要的修改点就需要很长时间，更别提修改带来的不可预知的影响面。

归根到底在于系统架构不清晰，划分出来的模块内聚度低、高耦合，导致代码不好复用、不好扩展、不好运维。

# 咋解决耦合？
第一种解决方案：按照演进式设计的理论，让系统的设计随着业务的演进而增长。这当然是可行的，工程实践中的重构、持续集成就可以对付各种混乱问题。问题在于，只是没有章法的、小范围的代码重构，很难具备通用型，容易变成了重构者的自娱自乐，代码继续腐败，重新重构…… 无休止的循环

第二种解决方案：重新抽象，如何抽象，DDD 建议我们双管齐下：1、分治（实体对象、值对象、聚合根）；2、分层（展示、应用、领域、通用）

# 咋做分治
核心概念三句话:

- 聚合根：一组相关对象的集合，作为一个整体被外界访问。聚合根的 ID 全局唯一
- 实体：有生命周期，有状态，通过 ID 进行唯一标识
- 值对象：属性，配合描述实体的状态

核心关系一句话：

通过聚合根来引用实体，挂载值对象，对外屏蔽内部的实体逻辑

talk is cheap，show me the code

```
 //聚合根
 class Order {
     public String id;//订单ID，全局唯一
     public Address customerAddress;//配送地址
     public List<Item> items;//商品信息
     public Pay pay;//支付信息
     public LogisticsDetail logisticsDetail;//物流信息
     public Pingjia pingjia;//评价信息
 }
 
 //实体
 class Item {
     public Long id; //商品ID，实体主键，Order内唯一
     public String name;//商品名
     public float price;//价格
     public int count;//数量
 }
 
   //实体
 class Pay {
     public Long id; //支付ID，实体主键，Pay内唯一
     public String source;//支付方式
     public int currency;//币种
     public float total;//价格
 }
 
  //实体
 class LogisticsDetail {
     public Long id; //物流ID，实体主键，LogisticsDetail内唯一
     public int cpCode;//物流公司
     public String mailNo;//物流单
     public float status;//当前状态
 }
 
 //实体
 class Pingjia {
     public Long id; //评价ID，实体主键，Pingjia内唯一
     public String desc;//描述
     public byte[] image;//图片
 }
 
 //值对象
 class Address{
     public String province;//省
     public String city;//市
     public String county;//区
 }
 ```
 
可以看到，通过业务限界，DDD 将大型复杂的 DO 分解为若干简单的 DO，从而保证 DO 的扩展性、灵活性。具体到 DB 层面，需要五张表：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019111923481915.png)

但如果业务没有这么复杂，我依然推荐 DO 的 4 要素设计：基础要素、核心要素、扩展要素以及冗余要素。就拿 open 店面绑定的 App 设计举例：

- 基础要素：createdTime、modifyTime
- 核心要素：developerId、appId、businessId、ePoiId、poiId
- 扩展要素：开发者的联系电话、店面的联系电话
- 冗余要素：feature

于此，我们可以归纳出领域模型设计的一般步骤：

- 根据需求划分出初步的业务域
- 识别出哪些领域是实体，哪些是值对象
- 对实体、值对象进行聚合，划分出聚合根
- 工程实践中检验模型的合理性，倒推模型中不足的地方并重新归纳

这里多说一句，如果你读过金字塔原理的话，会发现思维方式分为两种：归纳性思维和演绎性思维。人的原始思维方式是演绎性的，这就决定了我们经常基于流程去思考问题，而面向对象建模恰恰需要的是归纳性思维。这也是着重需要自我训练的地方。

# 咋做分界
## 模块
首先需要划分模块。模块（Module）是 DDD 中明确提到的分层前置手段，在工程实践中，较为常见的模块策略有两种：

技术职责分包：

- client：富客户端，thrift 接口划分在这一层，亦用于未来组织前置缓存、业务无关的通用校验等功能
- gateway：网关层，业务鉴权、请求过滤，并将请求对内映射成统一的 event 事件
- common：定义 core 和 client 所共用的内容
- core：业务服务的实现
- qatest：单元测试用例
- web：https 请求接口、业务自测 curl 接口

业务职责分包：（就拿开放平台 open 举例）

- open-base-client：兄弟系统的业务 rpc
- open-erp-client：erp 开发者 rpc
- open-pushrecord-client：推送订单 rpc
- open-web：外网请求

复杂系统建议采用技术分包策略，提高模块代码的复用性。

## 分层
然后我们再来讲模块内的分层，这里特指 core 模块的内部分层。DDD 描述了几个分层概念，分别是：防腐层、服务层、资源库、领域对象、基础层。

如代码中所示，一般的工程中包的组织方式为 {com. 公司名。组织架构。业务。上下文.*}，这样的组织结构能够明确的将一个上下文限定在包的内部。

- com.open.adapter.* ：防腐层（适配层，DO 转义）
- com.open.service.* ：服务层（逻辑流转层）
- com.open.tair.* ：资源库（分布式缓存、本地缓存）
- com.open.dao.* ：资源库（关系型、非关系型）
- com.open.domain.* ：领域对象（DO）
- com.open.util.* ：基础层（工具层）
这里着重释义一下防腐层，该层主要是将外部系统 DO 转义成本系统 DO，避免外部 DO 一旦发生变化，本系统改动范围过大的情况，收敛影响面。

```
  //接口层
  public class Consume {  
      public BaseResultDTO consumeMessage(OrderDetailMessage orderDetailMessage ) throws Exception, Throwable{
        OpenOrderDO openOrderDO = transferAdapter.orderDetailMessage2openOrderDO(orderDetailMessage);//调用防腐层
        openOrderService.handleOrderMeasage(openOrderDO);//调用服务层
      }
  }
  
  //防腐层
 public class TransferAdapter {
      public OpenOrderDO orderDetailMessage2openOrderDO(OrderDetailMessage orderDetailMessage){//转换DO
        OpenOrderDO openOrderDO = new OpenOrderDO();
        openOrderDO.setTradeId(orderDetailMessage.getTradeId());
        openOrderDO.setOrderSource(orderDetailMessage.getOrderSource());
        openOrderDO.setBuyerUid(orderDetailMessage.getBuyerId());
        openOrderDO.setSellerUid(orderDetailMessage.getSellerId());
        return openOrderDO;
      }
  }
```
# 咋落地
落地就是模型到代码的转换，核心是保证模型和代码的一致性。实际情况下，由于没有好的保持模型和代码一致的办法，很多系统往往开始搞的不错，慢慢就不一致了，也就逐渐烂掉了。

落地的方式，理论上有三种：

1. 给出架构设计，开发人员负责落地
2. 给出架构设计，给出全部实现
3. 给出架构设计，给出核心代码，开发人员去做扩展、实现

1 不可控，2 不现实。推荐 3 的做法：架构师给出设计方案，并给出骨干实现，开发人员有了可类比的代码，就能够比较准确的去做功能开发。这比空讲要有效的多。

此外，落地中还要注意三点：

1. 建立统一语言，避免认知偏差。客户、产品经理、架构师、开发人员在需求理解上的偏差，是后期返工的一大来源
2. 频繁沟通，迭代共识。 由于周期较短，即使发现了实现偏差，也能及时纠偏
3. 快速反馈。开发人员模糊地带主动询问架构师，尽可能保证模型与实现的一致

# 个人感悟
DDD 其实是面向对象方法论的一个升华。我们回头来看它，无外乎是通过划分领域（聚合根、实体、值对象）、领域行为封装到领域对象（充血模式）、内外交互封装到防腐层、职责封装到对应的模块和分层，从而实现了高内聚低耦合 —— 这也是它最精华的部分。

那么 DDD 的各个概念重要么？并不重要。概念掌握确实有助于提高我们的业务思考能力，但 DDD 强调的是理论结合实践，没有经过实战考验，都是纸上谈兵。经验丰富的开发人员，即便没有听说过 DDD，其思想也往往与 DDD 相符。我更建议的是：不要受限于对概念的掌握，而要透过层层表象思考它的本质，追求设计原则与实践方法的融汇贯通。只有如此，才能针对不同的场景灵活地运用 DDD，而非生搬硬套。毕竟，设计总是如此，即使前人已经总结了许多的方法，也不能像数学公式那样套用得到准确无误的结果。它不存在唯一的答案。

经验有限，我对 DDD 的理解难免会有不足之处，欢迎大家共同探讨，共同提高。