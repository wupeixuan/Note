一般在表设计的时候，都会在表中添加一些系统字段，比如 `create_time`、`update_time `等。

![](https://wupx-1256189981.file.myqcloud.com/img/202012/30/1609335594.png)

阿里巴巴开发手册中也有这样的提示，如果对于这些公共字段可以进行统一处理，不需要每次进行插入或者更新操作的时候 `set` 一下，就可以提高开发效率，解放双手。

## 加入依赖

下面就通过 MyBatis Plus 来完成字段自动填充，首先加入 MyBatis Plus 依赖：

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.1</version>
</dependency>
```

## 创建实体类，添加填充注解

创建一个实体类，然后在需要自动填充的属性上加注解 `@TableField(fill = FieldFill.INSERT)`、`@TableField(fill = FieldFill.INSERT_UPDATE)` 等注解。

```
@Data
@TableName("user")
public class UserEntity extends BaseEntity {
    private static final long serialVersionUID = 1L;
    /**
     * 主键
     */
    @TableId(value = "id", type = IdType.ASSIGN_ID)
    private Long id;
    /**
     * 姓名
     */
    @TableField("name")
    private String name;
    /**
     * 年龄
     */
    @TableField("age")
    private Integer age;
    /**
     * 邮件
     */
    @TableField("email")
    private String email;
    /**
     * 创建时间
     */
    @TableField(value = "create_time", fill = FieldFill.INSERT)
    public Date createTime;
    /**
     * 修改时间
     */
    @TableField(value = "modify_time", fill = FieldFill.INSERT_UPDATE)
    public Date modifyTime;
}
```

其中 fill 属性为字段自动填充策略，可选的参数如下所示：

```
public enum FieldFill {
    /**
     * 默认不处理
     */
    DEFAULT,
    /**
     * 插入填充字段
     */
    INSERT,
    /**
     * 更新填充字段
     */
    UPDATE,
    /**
     * 插入和更新填充字段
     */
    INSERT_UPDATE
}
```

就直接创建一个 Mapper，来便于测试：

```
@Mapper
public interface UserMapper extends BaseMapper<UserEntity> {
}
```

## 实现元对象处理器接口

MyBatis Plus 版本不同，实现方式可能会有些许不同，在 3.4.1 版本是实现 MetaObjectHandler 接口，低版本可能是继承 MetaObjectHandler 抽象类，来实现对应的方法。

下面为实现插入和更新数据的字段填充逻辑，在插入对象时，对创建时间 `createTime` 和修改时间 `modifyTime` 字段自动填充为当前时间，在更新对象时，将修改时间 `modifyTime` 修改为最新时间。

```
@Component
public class CommonMetaObjectHandler implements MetaObjectHandler {
    /**
     * 创建时间
     */
    private static final String FIELD_SYS_CREATE_TIME = "createTime";
    /**
     * 修改时间
     */
    private static final String FIELD_SYS_MODIFIED_TIME = "modifyTime";

    /**
     * 插入元对象字段填充（用于插入时对公共字段的填充）
     *
     * @param metaObject 元对象
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        Date currentDate = new Date();
        // 插入创建时间
        if (metaObject.hasSetter(FIELD_SYS_CREATE_TIME)) {
            this.strictInsertFill(metaObject, FIELD_SYS_CREATE_TIME, Date.class, currentDate);
        }
        // 同时设置修改时间为当前插入时间
        if (metaObject.hasSetter(FIELD_SYS_MODIFIED_TIME)) {
            this.strictUpdateFill(metaObject, FIELD_SYS_MODIFIED_TIME, Date.class, currentDate);
        }
    }

    /**
     * 更新元对象字段填充（用于更新时对公共字段的填充）
     *
     * @param metaObject 元对象
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        this.setFieldValByName(FIELD_SYS_MODIFIED_TIME, new Date(), metaObject);
    }
}
```

其中，默认填充策略为默认有值不覆盖，如果提供的值为 `null` 也不填充。如果默认填充策略不满足，可以重写 `strictFillStrategy` 方法以满足自己的需求。

## 测试字段自动填充

编写测试类来检验是否在插入和更新操作时，是否会自动填充响应的字段。

```
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest
public class AutoFillTest {
    @Resource
    private UserMapper userMapper;
    @Test
    public void test() throws InterruptedException {
        UserEntity user = new UserEntity();
        user.setName("wupx");
        user.setAge(18);
        user.setEmail("wupx@qq.com");
        userMapper.insert(user);
        Long id = user.getId();
        UserEntity beforeUser = userMapper.selectById(id);
        log.info("before user:{}", beforeUser);
        Assert.assertNotNull(beforeUser.getCreateTime());
        Assert.assertNotNull(beforeUser.getModifyTime());
        beforeUser.setAge(19);
        Thread.sleep(1000L);
        userMapper.updateById(beforeUser);
        log.info("query user:{}", userMapper.selectById(id));
        // 清除测试数据
        userMapper.deleteById(id);
    }
}
```

启动测试类，通过日志可以看出来：

```
before user:UserEntity(id=1346071927831134210, name=wupx, age=18, email=wupx@qq.com, createTime=Mon Jan 04 20:32:11 CST 2021, modifyTime=Mon Jan 04 20:32:11 CST 2021)
query user:UserEntity(id=1346071927831134210, name=wupx, age=19, email=wupx@qq.com, createTime=Mon Jan 04 20:32:11 CST 2021, modifyTime=Mon Jan 04 20:32:13 CST 2021)
```

第一次插入对象的时候，创建时间和修改时间都自动填充了，当修改对象的时候，修改时间也相应的进行了更新。

另外可以将公共字段封装到公共类中，比如 `BaseEntity`：

```
@Data
public class BaseEntity {
    /**
     * 主键
     */
    @TableId(value = "id", type = IdType.ASSIGN_UUID)
    private Long id;
    /**
     * 创建时间
     */
    @TableField(value = "create_time", fill = FieldFill.INSERT)
    private Date createTime;
    /**
     * 修改时间
     */
    @TableField(value = "modify_time", fill = FieldFill.INSERT_UPDATE)
    private Date modifyTime;
}
```

经过测试，也是可以完成公共字段的自动填充，大家也可以在项目中这样搞下，可以减少每次插入或者更新时的 `set` 操作。

# 总结

本文的完整代码在 `https://github.com/wupeixuan/SpringBoot-Learn` 的 `mybatis-plus-auto-fill-metainfo` 目录下。

你有没有经常需要去设置公共字段的烦恼呢，如果有这种情况，可以通过这种方式来解决下。

**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考
>
> https://github.com/wupeixuan/SpringBoot-Learn
>
> https://baomidou.com/guide/auto-fill-metainfo.html