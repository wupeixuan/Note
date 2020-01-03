fastjson是阿里巴巴的开源JSON解析库，它可以解析JSON格式的字符串，支持将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。

序列化：


```
String jsonString = JSON.toJSONString(obj);
```

反序列化：


```
VO vo = JSON.parseObject("...", VO.class);
```

## 实例
```
import com.alibaba.fastjson.JSON;

/**
 * @author wpx
 */
public class Demo {

    public static void main(String[] args) {
        User user = new User();
        user.setAge(24);
        user.setName("武培轩");
        String s = JSON.toJSONString(user);
        System.out.println(s);

        String jsonString = "{\"age\":24,\"birthdate\":19941030,\"name\":\"武培轩\",\"old\":false,\"salary\":5000.00}";
        User user1 = JSON.parseObject(jsonString, User.class);
        System.out.println(user1.getName());
    }
}

```

输出
```
{"age":24,"birthdate":0,"name":"武培轩","old":false,"salary":0.0}
武培轩

```