最近在项目开发中有向使用者发送报警通知的功能，其中报警媒介就包括邮件，这篇文章就简单介绍了 Spring Boot 如何快速集成实现邮件发送。

通常在实际项目中，也有其他很多地方会用到邮件发送，比如通过邮件注册账户/找回密码，通过邮件发送订阅信息等等。

### 开发前准备

下面以 QQ 邮箱为例，其他的邮箱的配置也大同小异。

登录 QQ 邮箱，点击**设置->账户**，开启**IMAP/SMTP服务**，并生成**授权码**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812223850699.png)

准备工作做好后，就开始进行项目实战吧！

## Spring Boot 集成邮件发送

Spring Boot 集成邮件发送主要分为以下三步：

1. 加入依赖
2. 配置邮件
3. 演示邮件发送

### 加入依赖

首先创建一个 Spring Boot 项目，然后在 `pom.xml` 加入如下依赖（其中 `thymeleaf` 是为了发送模版邮件）：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### 创建邮件配置

在配置文件 `application.properties` 中配置邮件的相关参数，具体内容如下：

```
# 使用 smtp 协议
spring.mail.protocol = smtp
spring.mail.host = smtp.qq.com
spring.mail.port = 587
spring.mail.username = wupx@qq.com
# 授权码
spring.mail.password = yourauthorizationcode
spring.mail.test-connection = false
spring.mail.properties.mail.smtp.auth = false
spring.mail.properties.mail.debug = false
spring.mail.properties.mail.mime.splitlongparameters = false
spring.mail.default-encoding = UTF-8
```

其中指定了邮件的协议、端口以及邮件账户和授权码等。

### 服务类

在这里主要介绍发送简单邮件、模板等操作，在 `service` 包下创建 `Mailervice` 类。

#### 发送简单邮件

首先注入 `JavaMailSender`，然后构造 `SimpleMailMessage` ，调用 `javaMailSender` 的 `send` 方法就可以完成简单的邮件发送，具体代码如下所示：

```
public void sendSimpleMail(String from, String to, String subject, String text) {
	SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
	// 发件人
	simpleMailMessage.setFrom(from);
	// 收件人
	simpleMailMessage.setTo(to);
	// 邮件主题
	simpleMailMessage.setSubject(subject);
	// 邮件内容
	simpleMailMessage.setText(text);
	javaMailSender.send(simpleMailMessage);
}
```

编写对应的 `Controller` 层，代码如下：

```
@GetMapping("/sendSimpleMail")
public void sendSimpleMail() {
	mailService.sendSimpleMail("wupx@qq.com",
			"huxy@qq.com",
			"欢迎关注微信公众号「武培轩」",
			"感谢你这么可爱，这么优秀，还来关注我，关注了就要一起成长哦~~回复【资料】领取优质资源！");
}
```

调用发送简单邮件接口后，就可以在 QQ 邮箱中收到邮件：

![](https://img-blog.csdnimg.cn/20200812231803784.png)

#### 发送复杂邮件

虽然简单的纯文本邮件已经基本够用了，但是有的时候，也需要很多样式来丰富邮件，接下来就来演示下发送复杂邮件（文本+图片+附件）。

首先使用 `javaMailSender` 创建一个 `MimeMessage` 实例，用 `MimeMessageHelper` 设置开启内嵌和附件功能，然后通过 `addInline` 方法嵌入图片（其中 `logo` 需要与 `text` 中的 `cid` 对应起来），`addAttachment` 方法添加附件，具体代码如下：

```
public ResponseEntity<String> sendMimeMail(String from, String to, String subject, String text) {
	MimeMessage mimeMessage = javaMailSender.createMimeMessage();
	try {
		MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
		helper.setFrom(from);
		helper.setTo(to);
		helper.setSubject(subject);
		// 设置邮件内容，第二个参数设置是否支持 text/html 类型
		helper.setText(text, true);
		helper.addInline("logo", new ClassPathResource("img/logo.jpg"));
		helper.addAttachment("logo.pdf", new ClassPathResource("doc/logo.pdf"));
		javaMailSender.send(mimeMessage);
		return ResponseEntity.status(HttpStatus.CREATED).body("发送成功");
	} catch (MessagingException e) {
		e.printStackTrace();
		return ResponseEntity.status(HttpStatus.NOT_FOUND).body("e.getMessage()");
	}
}
```

编写对应的 `Controller` 层，代码如下：

```
@GetMapping("/sendMimeMail")
public ResponseEntity<String> sendMimeMail() {
	return mailService.sendMimeMail("wupx@qq.com",
			"huxy@qq.com",
			"欢迎关注微信公众号「武培轩」",
			"<h3>感谢你这么可爱，这么优秀，还来关注我，关注了就要一起成长哦~~</h3><br>" +
					"回复【资料】领取优质资源！<br>" +
					"<img src='cid:logo'>");
}
```

调用发送复杂邮件接口后，就可以在 QQ 邮箱中收到邮件：

![](https://img-blog.csdnimg.cn/2020081300174940.png)

#### 发送模板邮件

Java 的模板引擎有许多，在这里使用的是 `Thymeleaf`，注入 `TemplateEngine`，使用它来解析模版，然后将返回的字符串作为内容发送，具体代码如下：

```
public ResponseEntity<String> sendTemplateMail(String from, String to, String subject, Context context) {
	MimeMessage message = javaMailSender.createMimeMessage();
	try {
		MimeMessageHelper helper = new MimeMessageHelper(message, true);
		helper.setFrom(from);
		helper.setTo(to);
		helper.setSubject(subject);
		// 解析邮件模板
		String text = templateEngine.process("mailTemplate", context);
		helper.setText(template, true);
		javaMailSender.send(message);
		return ResponseEntity.status(HttpStatus.CREATED).body("发送成功");
	} catch (Exception e) {
		e.printStackTrace();
		return ResponseEntity.status(HttpStatus.NOT_FOUND).body("e.getMessage()");
	}
}
```

编写对应的 `Controller` 层，代码如下：

```
@GetMapping("/sendTemplateMail")
public ResponseEntity<String> sendTemplateMail() {
	Context context = new Context();
	context.setVariable("username", "武培轩");
	return mailService.sendTemplateMail("wupx@qq.com",
			"huxy@qq.com",
			"欢迎关注微信公众号「武培轩」",
			context);
}
```

调用发送模板邮件接口后，就可以在 QQ 邮箱中收到邮件：

![](https://img-blog.csdnimg.cn/20200813005301172.png)

# 总结

本文的完整代码在 `https://github.com/wupeixuan/SpringBoot-Learn` 的 `mail` 目录下。

Spring Boot 集成邮件发送还是比较简单的，大家可以下载项目源码，自己在本地运行调试这个项目，更好地理解如何在 Spring Boot 中开发邮件服务。

**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考
>
> https://github.com/wupeixuan/SpringBoot-Learn
>