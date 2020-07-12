最近因为项目中需要存储很多的图片，不想存储到服务器上，因此就直接选用阿里云的对象服务（Object Storage Service，简称 OSS）来进行存储，本文将介绍 Spring Boot 集成 OSS 的一个完整过程。

那么 OSS 是什么呢？

简而言之，OSS 是一种海量、安全、低成本、高可靠的云存储服务。

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

然后在 config 包下创建 `OSSConfiguration` 类，会从配置文件中读取到对应的参数。

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

在这里主要介绍 OSS 的上传、下载、删除、查看 URL 等简单操作，在 `service` 包下创建 `OSSService` 类。

#### 上传文件

```
public String uploadFile(MultipartFile file, String storagePath) {
    String fileName = "";
    try {
        // 创建一个唯一的文件名，类似于id，就是保存在OSS服务器上文件的文件名
        fileName = UUID.randomUUID().toString();
        InputStream inputStream = file.getInputStream();
        // 设置对象
        ObjectMetadata objectMetadata = new ObjectMetadata();
        // 设置数据流里有多少个字节可以读取
        objectMetadata.setContentLength(inputStream.available());
        objectMetadata.setCacheControl("no-cache");
        objectMetadata.setHeader("Pragma", "no-cache");
        objectMetadata.setContentType(file.getContentType());
        objectMetadata.setContentDisposition("inline;filename=" + fileName);
        fileName = storagePath + "/" + fileName;
        // 上传文件
        PutObjectResult result = ossClient.putObject(ossConfiguration.getBucketName(), fileName, inputStream, objectMetadata);
        log.info("result:{}", result);
    } catch (IOException e) {
        log.error("Error occurred: {}", e.getMessage(), e);
    }
    return fileName;
}
```

#### 判断文件是否存在

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

```
public void exportFile(OutputStream os, String objectName) {
    // ossObject包含文件所在的存储空间名称、文件名称、文件元信息以及一个输入流。
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

#### 删除文件

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

```
public URL getSingeNatureUrl(String filename, int expSeconds) {
    if (expSeconds == 0) {
        expSeconds = 3600 * 1000;
    } else {
        expSeconds = expSeconds * 1000;
    }
    GeneratePresignedUrlRequest request = new GeneratePresignedUrlRequest(ossConfiguration.getBucketName(), filename);
    Date expiration = new Date(System.currentTimeMillis() + expSeconds);
    request.setExpiration(expiration);
    URL url = null;
    try {
        url = ossClient.generatePresignedUrl(ossConfiguration.getBucketName(), filename, expiration);
    } catch (ClientException e) {
        log.error("Error occurred: {}", e.getMessage(), e);
    }
    return url;
}
```

# 总结

本文的完整代码在 `https://github.com/wupeixuan/SpringBoot-Learn` 的 `oss` 目录下。


**最好的关系就是互相成就**，大家的**点赞、在看、分享、留言**就是我创作的最大动力。

> 参考
>
> https://github.com/wupeixuan/SpringBoot-Learn