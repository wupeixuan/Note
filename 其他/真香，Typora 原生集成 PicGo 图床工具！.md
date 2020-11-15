用 `markdown` 写作的话，Typora 是不错的选择，所见即所得，用得很舒服，但是在粘贴图片的时候只是把图片保存到本地，如果把 `md` 文件复制到其他平台或者网页时候会出现下面的情况，因此这篇文章主要讲下使用 Typora 结合 PicGo 实现图片自动上传，再也不用担心图片失效了。

![图片失效](https://wupx-1256189981.file.myqcloud.com/img/202011/04/1604478704.png)

## 安装 Typora 和 PicGo

首先安装 Typora，如果本来就有安装，需要注意版本需要在 0.9.84 以上，因为在这个版本开始支持通过 PicGo 上传图片，如果版本比较低，直接进行更新就行了。

![](https://wupx-1256189981.file.myqcloud.com/img/202011/04/1604491585.png)

再下载 [PicGo](https://molunerfinn.com/PicGo/)，可以在 `https://github.com/Molunerfinn/PicGo/releases` 进行下载，要记得安装路径，因为等会需要在 Typora 中需要进行配置。

然后打开**偏好设置->图像**，在插入图片时执行上传图片操作，然后在上传服务设定部分，选择已下载的 PicGo 的路径。

![](https://wupx-1256189981.file.myqcloud.com/img/202011/04/1604488620.png)

## 设置 PicGo 图床

PicGo 支持很多图床，比如SM.MS图床、腾讯云COS、阿里云OSS等，目前支持的图床如下：

![](https://wupx-1256189981.file.myqcloud.com/img/202011/04/1604491890.png)

现在以腾讯云COS为例，配置图床，



先开通腾讯云COS，然后进入图像存储页面，创建存储桶，把访问权限设置为**公有读私有写**。

再点击右上角，点击**访问管理->访问密钥->API密钥管理->新建密钥**，然后把 `SecretId` 和 `SecretKey` 记下。

![](https://wupx-1256189981.file.myqcloud.com/img/202011/04/1604492558.png)

然后在 PicGo 中填写对应的信息，至此图床就设置完成了，就可以实现在 Typora 中 `Ctrl+V` 就可以把图片上传到腾讯云COS中了。

如果想要使用其他图床，可以在 `https://picgo.github.io/PicGo-Doc/zh/guide/` 中进行查看。

### 自定义命名

另外如果觉得把所有图片都放在同一个路径下会比较乱，可以在插件设置中安装 `picgo-plugin-rename-file`，支持自定义命名。（安装插件需要 `npm` 环境，先去`https://nodejs.org/zh-cn/`下载）

![](https://wupx-1256189981.file.myqcloud.com/img/202011/04/1604493003.png)

这样就会自动按照年月/日/文件这样去上传文件，便于后期的管理和维护。

# 总结

大家可以根据自己的喜好选择图床进行存储，还没有使用的小伙伴可以搞起来！