在搭建网站的时候，往往会加载很多的图片，如果都从 Tomcat 服务器来获取静态资源，这样会增加服务器的负载，使得服务器运行 速度非常慢，这时可以使用 Nginx 服务器来加载这些静态资源，这样就可以实现负载均衡，为 Tomcat 服务器减压了。这篇文章，我们将一起去使用 Nginx 去搭建静态资源 web 服务器。

首先我把构建的 hexo 博客文件放在 Nginx 目录下，目录结构如下：

![](https://img-blog.csdnimg.cn/20191103172223860.png)

再修改 Nginx 配置文件 nginx.conf 中的 server：
```
server {
        listen 80;
        server_name localhost;

        location / {
                alias blog/;
        }  
    }
```

其中 `location /` 表示所有的请求，一般我们通过 root 和 alias 来指定访问的目录。root 相对来说有个问题，会把 url 中的一些路径带到我们的文件目录中来，所以一般使用 alias。

修改好配置文件后，执行 `nginx -s reload` 重启 nginx 服务，在浏览器中输入 `localhost/` 就可以访问了，如图所示：

![](https://img-blog.csdnimg.cn/20191103173901214.png)

此外还可以开启 gzip 压缩，服务器压缩，浏览器解压。压缩和解压减少的是中间网络传输的消耗。

修改 nginx.conf：

```
gzip on;
gzip_min_length 1;
gzip_comp_level 2;
gzip_types text/plain application/x-javascript text/css application/pdf application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
```
其中，`gzip_min_length` 表示小于此大小则不压缩，`gzip_comp_level` 表示压缩等级，`gzip_types` 表示压缩类型。

通过 url 访问，查看消息头就可以看到已经开启 gzip 压缩了：`Content-Encoding: gzip`

![](https://img-blog.csdnimg.cn/20191103173743524.png)

使用 gzip 压缩之后，静态资源的传输效率会提升很多。

还可以打开目录浏览功能，修改 nginx 的配置文件，添加 `autoindex on;`
```
server {

    listen 80;
    server_name localhost;

    location / {
           alias blog/;
           autoindex on;
    }
}
```
修改后，重启 nginx，以目录结构中的 images 目录为例，访问 url：`localhost/images/`，展示情况如下图：

![](https://img-blog.csdnimg.cn/20191103174541702.png)

为了防止访问大文件抢走带宽，可通过设置访问资源时传输的速度来限制访问的文件大小。
```
server {
    listen 80;
    server_name localhost;

    location / {
            alias blog/;
            autoindex on;
            set $limit_rate 100K;
    }
}
```
其中 `set $limit_rate 100K;` 表示每秒传输速度限制在 100K 大小。

> 参考
> 
> http://nginx.org/en/docs/http/ngx_http_core_module.html