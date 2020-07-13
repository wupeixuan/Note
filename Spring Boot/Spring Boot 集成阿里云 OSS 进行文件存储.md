最近因为项目中需要存储很多的图片，不想存储到服务器上，因此就直接选用阿里云的对象服务（`Object Storage Service`，简称 OSS）来进行存储，本文将介绍 Spring Boot 集成 OSS 的一个完整过程。

那么 OSS 是什么呢？

简而言之，OSS 是一种海量、安全、低成本、高可靠的云存储服务。

关于 OSS 的知识就不再这里赘述了，大家可以自行学习下。

## 开通 OSS

首先需要在阿里云控制台开通 OSS，然后需要创建存储空间（Bucket），我这里命名为 `wupx-img`。

![创建 Bucket](https://img-blog.csdnimg.cn/20200712175636178.png)

> 如果之前没有创建过 AccessKey，鼠标移到右上角的账号后点击 AccessKey 管理，然后创建就可以了。
>
> 或者直接输入 https://ak-console.aliyun.com/#/ 来进行密钥的创建和查看。

![AccessKey 管理](https://img-blog.csdnimg.cn/20200712180550514.png)

准备工作做好后，就开始进行项目实战吧！

## Spring Boot 集成 OSS

Spring Boot 集成 OSS 主要分为以下三步：

1. 加入 OSS 依赖
2. 配置 OSS
3. 演示 OSS 基本操作

### 加入依赖

首先创建一个 Spring Boot 项目，然后在 `pom.xml` 加入如下依赖集成 OSS：

```
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.10.2</version>
</dependency>
```

### 创建 OSS 配置

在配置文件 `application.properties` 中配置 OSS 的相关参数，具体内容如下：

```
#访问OSS的域名
aliyun.endpoint=http://oss-cn-beijing.aliyuncs.com
aliyun.accessKeyId=yourAccessKeyId
aliyun.accessKeySecret=yourAccessKeySecret
#管理所存储Object的存储空间名称
#aliyun.bucketName=yourBucketName
spring.servlet.multipart.max-request-size=10MB
spring.servlet.multipart.max-file-size=10MB
```

其中指定了 OSS 的访问域名、密钥以及存储空间 Bucket 的名称等。

然后在 config 包下创建 `OSSConfiguration` 类，会从配置文件中读取到对应的参数，并且把 ossClient 单例化。

```
@Configuration
@Component
public class OSSConfiguration {

    private volatile static OSS ossClient;

    private volatile static OSSClientBuilder ossClientBuilder;

    private static String endpoint;

    private static String accessKeyId;

    private static String accessKeySecret;

    @Value("${aliyun.bucketName}")
    private String bucketName;

    @Value("${aliyun.endpoint}")
    public void setEndpoint(String endpoint) {
        OSSConfiguration.endpoint = endpoint;
    }

    @Value("${aliyun.accessKeyId}")
    public void setAccessKeyId(String accessKeyId) {
        OSSConfiguration.accessKeyId = accessKeyId;
    }

    @Value("${aliyun.accessKeySecret}")
    public void setAccessKeySecret(String accessKeySecret) {
        OSSConfiguration.accessKeySecret = accessKeySecret;
    }

    public String getBucketName() {
        return bucketName;
    }

    @Bean
    @Scope("prototype")
    public static OSS initOSSClient() {
        if (ossClient == null) {
            synchronized (OSSConfiguration.class) {
                if (ossClient == null) {
                    ossClient = initOSSClientBuilder().build(endpoint, accessKeyId, accessKeySecret);
                }
            }
        }
        return ossClient;
    }

    public static OSSClientBuilder initOSSClientBuilder() {
        if (ossClientBuilder == null) {
            synchronized (OSSConfiguration.class) {
                if (ossClientBuilder == null) {
                    ossClientBuilder = new OSSClientBuilder();
                }
            }
        }
        return ossClientBuilder;
    }
}
```

### 服务类

在这里主要介绍 OSS 的上传、下载、删除、查看 URL 等简单操作，在 `service` 包下创建 `OSSService` 类，然后注入 `ossClient` 和 `ossConfiguration`。

#### 上传文件

首先来看下如何上传文件，首先通过 UUID 生成文件名，防止重复，再创建一个 `ObjectMetadata`，可以设置用户自定义的元数据以及 HTTP 头，比如内容长度，ETag 等，最后通过调用 `ossClient` 的 `putObject` 方法来完成文件上传，并返回文件名，具体代码如下所示：

```
public String uploadFile(MultipartFile file, String storagePath) {
    String fileName = "";
    try {
        fileName = UUID.randomUUID().toString();
        InputStream inputStream = file.getInputStream();
        ObjectMetadata objectMetadata = new ObjectMetadata();
        objectMetadata.setContentLength(inputStream.available());
        objectMetadata.setCacheControl("no-cache");
        objectMetadata.setHeader("Pragma", "no-cache");
        objectMetadata.setContentType(file.getContentType());
        objectMetadata.setContentDisposition("inline;filename=" + fileName);
        fileName = storagePath + "/" + fileName;
        // 上传文件
        ossClient.putObject(ossConfiguration.getBucketName(), fileName, inputStream, objectMetadata);
    } catch (IOException e) {
        log.error("Error occurred: {}", e.getMessage(), e);
    }
    return fileName;
}
```

编写对应的 `Controller` 层，调用上传文件接口后，在文件管理中可以看到文件已经上传成功了：

![上传文件](https://img-blog.csdnimg.cn/20200713222546198.png)

#### 获取文件列表

可以通过 `ListObjectsRequest` 构建请求参数，比如设置 Bucket 名称和列举文件的最大个数，然后调用 `ossClient` 的 `listObjects` 方法就可以获取到 `objectListing`，再获取文件的元信息，最后将文件名称返回，具体代码如下：

```
public List<String> listObjects() {
    ListObjectsRequest listObjectsRequest = new ListObjectsRequest(ossConfiguration.getBucketName()).withMaxKeys(200);
    ObjectListing objectListing = ossClient.listObjects(listObjectsRequest);
    List<OSSObjectSummary> objectSummaries = objectListing.getObjectSummaries();
    return objectSummaries.stream().map(OSSObjectSummary::getKey).collect(Collectors.toList());
}
```

#### 判断文件是否存在

判断文件是否存在，直接通过 `ossClient` 的 `doesObjectExist` 方法就可以进行判断，传入的参数为 Bucket 名称和文件名称，具体代码如下：

```
public boolean doesObjectExist(String fileName) {
    try {
        if (Strings.isEmpty(fileName)) {
            log.error("文件名不能为空");
            return false;
        } else {
            return ossClient.doesObjectExist(ossConfiguration.getBucketName(), fileName);
        }
    } catch (OSSException | ClientException e) {
        e.printStackTrace();
    }
    return false;
}
```

#### 下载文件

下载存储在 OSS 的文件，首先需要传入 Bucket 名称和文件名称调用 `ossClient` 的 `getObject` 方法获取 `ossObject`，`ossObject` 包含文件所在的存储空间名称、文件名称、文件元信息以及一个输入流，然后调用 `ossObject` 的 `getObjectContent` 方法获取输入流，然后进行文件的下载，具体代码如下：

```
public void exportFile(OutputStream os, String objectName) {
    OSSObject ossObject = ossClient.getObject(ossConfiguration.getBucketName(), objectName);
    // 读取文件内容
    BufferedInputStream in = new BufferedInputStream(ossObject.getObjectContent());
    BufferedOutputStream out = new BufferedOutputStream(os);
    byte[] buffer = new byte[1024];
    int lenght;
    try {
        while ((lenght = in.read(buffer)) != -1) {
            out.write(buffer, 0, lenght);
        }
        out.flush();
    } catch (IOException e) {
        log.error("Error occurred: {}", e.getMessage(), e);
    }
}
```

![](https://img-blog.csdnimg.cn/20200713225007225.png)

#### 删除文件

删除文件也比较简单，直接调用 `deleteObject` 方法，传入对应的 Bucket 名称和文件名称即可，具体代码如下：

```
public void deleteFile(String fileName) {
    try {
        ossClient.deleteObject(ossConfiguration.getBucketName(), fileName);
    } catch (Exception e) {
        log.error("Error occurred: {}", e.getMessage(), e);
    }
}
```

#### 查看 URL

获取文件的访问地址可以调用 `ossClient` 的 `generatePresignedUrl`，在调用的时候还需要设置过期时间，具体代码如下：

```
public String getSingeNatureUrl(String filename, int expSeconds) {
    Date expiration = new Date(System.currentTimeMillis() + expSeconds * 1000);
    URL url = ossClient.generatePresignedUrl(ossConfiguration.getBucketName(), filename, expiration);
    if (url != null) {
        return url.toString();
    }
    return null;
}
```

通过调用接口即可返回文件对应的 url 地址，我们通过 url 就可以访问图片，效果如下：

![](https://img-blog.csdnimg.cn/20200713233152781.png)

到此为止，OSS 的基本操作就简单介绍完了，大家可以多动手试试，不会的可以看下官方的帮助文档。

##### 跨域规则

阿里云 OSS 解决请求跨越问题：进入对应的 Bucket，然后依次点击权限管理->跨越设置->创建规则，然后填写上对应的规则，具体如下图所示：

![跨域规则](https://img-blog.csdnimg.cn/20200713220728125.png)

# 总结

本文的完整代码在 `https://github.com/wupeixuan/SpringBoot-Learn` 的 `oss` 目录下。

Spring Boot 结合 OSS 还是比较简单的，大家可以下载项目源码，自己在本地运行调试这个项目，更好地理解如何在 Spring Boot 中构建基于 OSS 的应用。

**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考
>
> https://github.com/wupeixuan/SpringBoot-Learn
>
> https://help.aliyun.com/document_detail/32008.html