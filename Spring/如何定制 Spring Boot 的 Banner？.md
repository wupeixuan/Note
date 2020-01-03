相信用过 Spring Boot 的朋友们一定在启动日志中见过类似如下的内容，比如在启动 Spring Boot 时，控制台默认会打印 Spring Boot Logo 以及版本信息，这是 Spring Boot 固定的还是可自定义的呢？

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.7.RELEASE)
```

答案是，Spring Boot 支持自定义 Banner，接下来本文将详细讨论如何定制 Banner 内容，首先来了解下 Banner 是如何出现的。

# Banner 是如何出现的？
初始 Banner 的代码是 SpringApplicationBannerPrinter 类，Spring Boot 默认寻找 Banner 的顺序是:
- 首先依次在 Classpath 下找文件 banner.gif，banner.jpg 和 banner.png，使用优先找到的
- 若没找到上面文件的话，继续 Classpath 下找 banner.txt
- 若上面都没有找到的话, 用默认的 SpringBootBanner，也就是上面输出的 Spring Boot Logo

一般是把 banner.* 文件放在 src/main/resources/ 目录下。

我们可以用属性 banner.location 设定 Spring Boot 在不同于 Classpath  下找以上 banner.txt 文件，banner.charset 设定 banner.txt 的字符集，默认为 UTF-8。属性 banner.image.location 用于指定寻找 banner.(gif|jpg|png) 文件的位置。

如果同时存在图片(如 banner.jpg) 和 banner.txt , 则它们会同时显示出来，先图片后文字，但同时存在多个图片 banner.(gif|jpg|png)，则只会显示第一张图片。

- 对于文本文件，Spring Boot 会将其直接输出。
- 对于图像文件（ `banner.gif` 、`banner.jpg` 或 `banner.png` ），Spring Boot 会将图像转为 ASCII 字符，然后输出。

# 变量

banner.txt 文件中还可以使用变量来设置字体、颜色、版本号。

| 变量  | 描述       |
| --- | --- |
| `${application.version}`| `MANIFEST.MF` 中定义的版本。如：`1.0`|
| `${application.formatted-version}`| `MANIFEST.MF` 中定义的版本，并添加一个 `v` 前缀。如：`v1.0`|
| `${spring-boot.version}`| Spring Boot 版本。如：`1.5.7.RELEASE`|
| `${spring-boot.formatted-version}`| Spring Boot 版本，并添加一个 `v` 前缀。如：`v1.5.7.RELEASE`|
| `${Ansi.NAME}` (or `${AnsiColor.NAME}`, `${AnsiBackground.NAME}`, `${AnsiStyle.NAME}`) | ANSI 颜色、字体 |
| `${application.title}`| `MANIFEST.MF` 中定义的应用名|

# 配置
`application.properties` 中与 Banner 相关的配置：

```
# banner 模式。有三种模式：console/log/off
# console 打印到控制台（通过 System.out）
# log - 打印到日志中
# off - 关闭打印
spring.main.banner-mode = off
# banner 文件编码
spring.banner.charset = UTF-8
# banner 文本文件路径
spring.banner.location = classpath:banner.txt
# banner 图像文件路径（可以选择 png,jpg,gif 文件）
spring.banner.image.location = classpath:banner.gif
used).
# 图像 banner 的宽度（字符数）
spring.banner.image.width = 76
# 图像 banner 的高度（字符数）
spring.banner.image.height =
# 图像 banner 的左边界（字符数）
spring.banner.image.margin = 2
# 是否将图像转为黑色控制台主题
spring.banner.image.invert = false
```

当然，也可以在 YAML 文件中配置，例如：

```
spring:
    banner:
        charset: UTF-8
        location: classpath:banner.txt
```

# 示例
新建 Spring Boot 项目（基于 Spring Boot 1.5.7）
```
package com.wupx.springboot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class BannerApplication {

	public static void main(String[] args) {
		SpringApplication.run(BannerApplication.class, args);
	}

}
```

在 Spring Boot 项目中的 `resources` 目录下添加 banner.txt 文件，内容如下：

```
${AnsiColor.BRIGHT_YELLOW}${AnsiStyle.BOLD}
__  _  ___________  ___
\ \/ \/ /\____ \  \/  /
 \     / |  |_> >    <
  \/\_/  |   __/__/\_ \
         |__|        \/
${AnsiColor.CYAN}${AnsiStyle.BOLD}
::  Java                 ::  (v${java.version})
::  Spring Boot          ::  (v${spring-boot.version})
${AnsiStyle.NORMAL}
```

启动 Spring Boot 应用后，控制台输出的 Banner 如下：

![logo](https://img-blog.csdnimg.cn/20191027203728557.png)

推荐几个生成字符画的网站，可以将生成的字符画放入这个 `banner.txt` 文件：

- http://www.network-science.de/ascii/
- http://patorjk.com/software/taag
- http://www.degraeve.com/img2txt.php

# 总结
默认 Spring Boot 会注册一个 `SpringBootBanner` 的单例 Bean，用来负责打印 Banner。

如果想完全个人定制 Banner，可以先实现 `org.springframework.boot.Banner#printBanner` 接口来自己定制 Banner。在将这个 Banner 通过 `SpringApplication.setBanner()` 方法注入 Spring Boot。

一般自定义 Spring Boot Banner 是企业/团队/项目的 Slogan。


> 参考
> 
> https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-banner