Git是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。 Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。 Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

## Git下载
直接在 https://git-scm.com/downloads 里面，下载最新版的Git，默认安装就可以了。

安装完成后，在开始菜单里找到 Git->Git Bash，点击后出现一个类似命令行窗口的东西，就说明Git安装成功。

## Git配置
在命令行中配置本地仓库的账号和邮箱
```
$ git config --global user.name "wupeixuan"  
$ git config --global user.email "wupx@missfresh.cn"  
```

为避免每次远程访问输入密码，使用ssh登陆。ssh是与本机信息绑定的，所以每台电脑需要单独生成。
```
$ ssh-keygen -t rsa -C "youremail@example.com"  
```

ssh现在只是在本地，需要在GitLab中备份，才能被验证。打开自己的GitLab，在My Profile中，点击Add Public Key，title随意。

key中的内容在本机C盘中，C:\Users\wpx\.ssh（你的账户下），里面有个.ssh文件夹，用文本文档打开id_rsa.pub，将里面的内容全部复制到key中，即可；

到此，基本配置完毕；我们需要获取GitLab上项目的地址，每个项目地址不同，一般在GitLab的Projects中，能找到跟你相关的所有项目，点开一个项目，就能看到项目地址，然后在Git Bash中输入：

```
$ git clone git@git.missfresh.cn:grampus/grampus-replenishment.git
```

将数据同步到本地，一般关联后，直接:
```
$ git pull  
```

即可完成项目的拉取

至此，我们完成了一个在GitLab上的项目，到本地的过程。

## Git常用命令
```txt
#初始化
$ git init

#从仓库克隆
$ git clone [url]

#将修改放入暂存区
$ git add

#将暂存区的改动提交到本地仓库
$ git commit

#移除文件(从暂存区移除)
$ git rm

#取消暂存
$ git reset HEAD <file>

#查看分支
$ git branch

#创建分支
$ git branch f_20180428_orderMigration

#将HEAD指针向后移动一位到原分支
git checkout HEAD^1

#取消对文件的修改
$ git checkout -- <file>

#切换分支
$ git checkout f_20180428_orderMigration

#创建+切换分支
$ git checkout -b f_20180428_orderMigration

#合并某分支到当前分支
$ git merge origin master

#删除分支
$ git branch -d f_20180428_orderMigration

#查看变更历史
$ git log

#检查文件状态
$ git status

#推送到远程仓库
$ git push origin master

#
$ git rebase
```

## Git解决冲突
首先拉取线上代码
`git pull origin master`，再合并主分支然后我合并主分支`git merge master`，然后通过`git status`查看冲突文件，对冲突文件进行修改，修改完成后添加到暂存区，`git add xxx.java`然后统一git commit将修改合并的文件添加到工作区，`git commit -m "修改"`，提交到远程仓库，`git push origin f_20180507_distributionHotGoods `。



## 统一git分支命名规范
```txt
feature功能分支命名规范：f_时间戳_功能，注意下划线不是中线-
正确实例：
f_20180326_orderMigration
fixbug bug修复分支命名规范：
x_时间戳_功能
正确实例：
x_20180326_orderMigration
```

## 小技巧
```txt
Git 命令别名
$ git config --global alias.co checkout
$ git config --global alias.br branch
$ git config --global alias.ci commit
$ git config --global alias.st status
```