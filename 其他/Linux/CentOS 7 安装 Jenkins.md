# Jenkins介绍

Jenkins是一个开源的支持自动化构建、部署等任务的平台。基本上可以说是持续集成（CI）、持续发布（CD）不可或缺的工具。

# 安装Java环境

[CentOS 7 安装 JAVA环境（JDK 1.8）](https://www.cnblogs.com/wupeixuan/p/11433922.html)

# 安装Git

`yum install git`

# 安装Maven

`yum install maven`

# 安装Jenkins

`yum install jenkins`

修改jenkins启动脚本

`vi /etc/init.d/jenkins`

修改candidates增加java可选路径：
```
candidates="
/usr/java/jdk1.8.0_181/bin/java
"
```

启动Jenkins并设置Jenkins开机启动

```
#重载服务（使修改的Jenkins启动脚本生效）
systemctl daemon-reload

#启动Jenkins服务
systemctl start jenkins

#启动Jenkins服务
systemctl stop jenkins

#将Jenkins服务设置为开机启动
#由于Jenkins不是Native Service，所以需要用chkconfig命令而不是systemctl命令
sudo /sbin/chkconfig jenkins on
```

# Jenkins 初始化

查询 Jenkins 默认密码
`cat /var/lib/jenkins/secrets/initialAdminPassword`

输入密码后，安装推荐的插件

添加管理员用户

配置 Jenkins URL

开始使用 Jenkins

启动 Jenkins

`service jenkins start`

关闭 Jenkins

`service jenkins stop`

查询 Jenkins 安装路径

`whereis jenkins`