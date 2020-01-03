# 报错情况
服务端写了一个对外的接口，传入 json 数据进行处理，其中该 json 带有日期，格式为 yyyy-MM-dd HH:mm:ss

调用该接口时报异常，异常如下：

```
com.fasterxml.jackson.databind.exc.InvalidFormatException: Can not deserialize value of type java.util.Date from String "2018-01-01 00:00:00": not a valid representation (error: Failed to parse Date value '2018-01-01 00:00:00': Can not parse date "2018-01-01 00:00:00": while it seems to fit format 'yyyy-MM-dd'T'HH:mm:ss.SSS', parsing fails (leniency? null))
 at [Source: [
    {
        "enterpriseId": "1-100DI2J",
        "projectId": "15001346284030848058",
        "type": 1,
        "livenessCount": 12,
        "livenessDate": "2019-10-18 00:00:00"
    }
]; line: 7, column: 25] (through reference chain: java.util.ArrayList[0]->com.wupx.mq.domain.ProductFunctionDetail["livenessDate"])
	at com.fasterxml.jackson.databind.exc.InvalidFormatException.from(InvalidFormatException.java:74)
	at com.fasterxml.jackson.databind.DeserializationContext.weirdStringException(DeserializationContext.java:1410)
	at com.fasterxml.jackson.databind.DeserializationContext.handleWeirdStringValue(DeserializationContext.java:926)
	at com.fasterxml.jackson.databind.deser.std.StdDeserializer._parseDate(StdDeserializer.java:819)
	at com.fasterxml.jackson.databind.deser.std.StdDeserializer._parseDate(StdDeserializer.java:788)
	at com.fasterxml.jackson.databind.deser.std.DateDeserializers$DateBasedDeserializer._parseDate(DateDeserializers.java:172)
	at com.fasterxml.jackson.databind.deser.std.DateDeserializers$DateDeserializer.deserialize(DateDeserializers.java:259)
	at com.fasterxml.jackson.databind.deser.std.DateDeserializers$DateDeserializer.deserialize(DateDeserializers.java:242)
	at com.fasterxml.jackson.databind.deser.SettableBeanProperty.deserialize(SettableBeanProperty.java:504)
	at com.fasterxml.jackson.databind.deser.impl.MethodProperty.deserializeAndSet(MethodProperty.java:104)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.vanillaDeserialize(BeanDeserializer.java:276)
	at com.fasterxml.jackson.databind.deser.BeanDeserializer.deserialize(BeanDeserializer.java:140)
	at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:287)
	at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:259)
	at com.fasterxml.jackson.databind.deser.std.CollectionDeserializer.deserialize(CollectionDeserializer.java:26)
	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:3814)
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:2877)
	at com.wupx.common.util.JsonUtils.fromJson(JsonUtils.java:117)
	at com.wupx.mq.handler.ProductHandler.doProcess(ProductHandler.java:29)
	at com.wupx.mq.consumer.TopicProductConsumer.processMap(TopicProductConsumer.java:45)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:180)
	at org.springframework.messaging.handler.invocation.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:112)
	at org.springframework.amqp.rabbit.listener.adapter.HandlerAdapter.invoke(HandlerAdapter.java:49)
	at org.springframework.amqp.rabbit.listener.adapter.MessagingMessageListenerAdapter.invokeHandler(MessagingMessageListenerAdapter.java:126)
	at org.springframework.amqp.rabbit.listener.adapter.MessagingMessageListenerAdapter.onMessage(MessagingMessageListenerAdapter.java:106)
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.doInvokeListener(AbstractMessageListenerContainer.java:848)
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.invokeListener(AbstractMessageListenerContainer.java:771)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.access$001(SimpleMessageListenerContainer.java:102)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$1.invokeListener(SimpleMessageListenerContainer.java:198)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.invokeListener(SimpleMessageListenerContainer.java:1311)
	at org.springframework.amqp.rabbit.listener.AbstractMessageListenerContainer.executeListener(AbstractMessageListenerContainer.java:752)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.doReceiveAndExecute(SimpleMessageListenerContainer.java:1254)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.receiveAndExecute(SimpleMessageListenerContainer.java:1224)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer.access$1600(SimpleMessageListenerContainer.java:102)
	at org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer$AsyncMessageProcessingConsumer.run(SimpleMessageListenerContainer.java:1470)
	at java.lang.Thread.run(Thread.java:748)
```

# 解决方法
Jackson 对 Date 默认转换格式才用的 yyyy-MM-dd'T'HH:mm:ss.SSS ，因此会出现转换异常。

实体类中日期属性添加下面的这个注解,自动转换格式，就不会报异常了。
```
    @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss")
    private Date livenessDate;
```