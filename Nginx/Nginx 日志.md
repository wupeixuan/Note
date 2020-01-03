# 用 GoAccess 实现可视化并实时监控 access 日志
安装 GoAccess
```
# 安装依赖
yum install geoip-devel ncurses ncurses-devel glib2-devel
# 下载
wget https://tar.goaccess.io/goaccess-1.3.tar.gz
# 解压
tar -xzvf goaccess-1.3.tar.gz
cd goaccess-1.3/
# 配置
./configure --enable-utf8 --enable-geoip=legacy
# 编译
make
# 安装
make install
```

配置 nginx
```
        location /report.html {
                alias /usr/local/nginx/html/report.html;
        }
```

重启 nginx 后，切换到 logs 目录下
`goaccess nginx.access.log -o ../html/report.html --real-time-html --time-format='%H:%M:%S' --date-format='%d%b%Y' --log-format=COMBINED`

打开 http://192.168.81.129:8080/report.html 就可看到监控日志
![GoAccess 监控日志](https://img2018.cnblogs.com/blog/1356806/201909/1356806-20190929194124468-958214679.png)

## 基于 OpenResty 用 Lua 语言实现简单服务
安装 OpenResty

```
wget https://openresty.org/download/openresty-1.15.8.2.tar.gz
tar -xzvf openresty-1.15.8.2.tar.gz 
cd openresty-1.15.8.2
./configure
# 编译
make
# 安装
make install
```

配置 nginx.conf 配置文件，加上下面这段代码
```
        location /lua {
                default_type 'text/html';
                content_by_lua 'ngx.say("User-Agent：", ngx.req.get_headers()["User-Agent"])';
        }
```
打开 http://192.168.81.129:8080/lua 就可获取 User-Agent 列表