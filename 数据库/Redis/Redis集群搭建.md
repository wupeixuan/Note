Redis、Ruby语言运行环境、Redis的Ruby驱动redis-3.2.2.gem、创建Redis集群的工具redis-trib.rb。使用redis-trib.rb工具来创建Redis集群，由于该文件是用ruby语言写的，所以需要安装Ruby开发环境，以及驱动redis-3.2.2.gem。

## 下载
- 下载Redis安装文件：https://github.com/MSOpenTech/redis/releases/，Redis提供msi和zip格式的下载文件，这里下载zip格式Redis-x64-3.2.100版本。
- 下载Ruby安装文件：http://dl.bintray.com/oneclick/rubyinstaller/rubyinstaller-2.2.4-x64.exe
- 下载Ruby环境下Redis的驱动：https://rubygems.org/gems/redis/versions/3.2.2，考虑到兼容性，这里下载的是3.2.2版本
- 下载Redis官方提供的创建Redis集群的ruby脚本文件redis-trib.rb，路径如下：https://raw.githubusercontent.com/MSOpenTech/redis/3.0/src/redis-trib.rb

## 安装Redis
将下载到的Redis-x64-3.2.100.zip解压即可，F:\RedisCluster\Redis-x64-3.2.100。 

安装Redis，并运行6个实例。

通过配置文件来启动6个不同的Redis实例，Redis默认端口为6379，这里使用了10001，10002，10003，10004，10005，10006来运行6个Redis实例。

./redis-server.exe --service-install  ./redis.10001.conf --service-name redis10001
./redis-server.exe --service-install  ./redis.10002.conf --service-name redis10002
./redis-server.exe --service-install  ./redis.10003.conf --service-name redis10003
./redis-server.exe --service-install  ./redis.10004.conf --service-name redis10004
./redis-server.exe --service-install  ./redis.10005.conf --service-name redis10005
./redis-server.exe --service-install  ./redis.10006.conf --service-name redis10006


./redis-server.exe --service-uninstall  ./redis.10001.conf --service-name redis10001
./redis-server.exe --service-uninstall  ./redis.10002.conf --service-name redis10002
./redis-server.exe --service-uninstall  ./redis.10003.conf --service-name redis10003
./redis-server.exe --service-uninstall  ./redis.10004.conf --service-name redis10004
./redis-server.exe --service-uninstall  ./redis.10005.conf --service-name redis10005
./redis-server.exe --service-uninstall  ./redis.10006.conf --service-name redis10006

./redis-server.exe --service-start --service-name redis10001
./redis-server.exe --service-start --service-name redis10002
./redis-server.exe --service-start --service-name redis10003
./redis-server.exe --service-start --service-name redis10004
./redis-server.exe --service-start --service-name redis10005
./redis-server.exe --service-start --service-name redis10006

./redis-trib.rb create --replicas 1 10.1.84.22:10001 10.1.84.22:10002 10.1.84.22:10003 10.1.84.22:10004 10.1.84.22:10005 10.1.84.22:10006
