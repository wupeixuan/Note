报错情况：Git 在推送项目时报错：fatal：The remote end hung up unexpectedly

问题原因：推送的文件太大。

解决方法一：

修改设置 git config 文件的 postBuffer 的大小。（设置为 500MB）

```
$ git config --local http.postBuffer 524288000
```

注：--local 选项指定这个设置只对当前仓库生效。

解决方法二：

直接修改本地仓库的 .config 文件。

1. 打开项目所在的目录
2. 设置显示隐藏文件
3. 编辑 config 文件

增加下面两行内容：

```
[http]
	postBuffer = 524288000
```