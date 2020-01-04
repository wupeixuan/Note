![idea](https://img-blog.csdnimg.cn/2020010414285181.png)

今天发现好多人的IDEA激活码都到期了，IDEA社区版又不能满足开发需求，因此写这篇IDEA的激活文章，希望对大家有用。

以下方法的破解文件的是永久破解的，不存在过期时间。

当然，有条件还是买正版授权较好，学生党也可以通过 edu 邮箱去申请激活码。

# 官网下载

先把之前版本的 IDEA 删除，然后去官网下载最新版的 IDEA。

下载地址：`https://www.jetbrains.com/idea/download/`

下载完成后，启动 IDEA 的安装包，完成安装。

# 激活 IDEA

安装完毕后，打开 IDEA，先勾选 `Evaluate for free` 免费试用 30 天，进去之后选择 Configure -> Edit Custom VM Options...，如下图：

![vm](https://img-blog.csdnimg.cn/20200104143636260.png)

打开以后在文本的最后，添加一行：

```
-javaagent:D:\SoftWare\IntelliJ IDEA 2019.3.1\jetbrains-agent.jar
```

注：路径为 jetbrains-agent.jar 的绝对路径，如我的路径为 IDEA 目录下的路径，路径中一定不能包含中文！！！

![激活](https://img-blog.csdnimg.cn/20200104144102325.png)

> 在公众号【武培轩】回复【idea】关键字获取破解补丁的网盘链接。
> 
> 里面包含两个文件：
> 
> 1：破解补丁文件
> 
> 2：激活教程


在修改完之后，关闭 IDEA，重新打开 IDEA（选择以管理员身份运行）。

打开 IDEA 后，选择 Configure -> Manage License。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020010414514387.png)

### 1. 选择 License Server

选择成功一般情况下，Server address 一栏就会出现 http://jetbrains-license-server

如果实在是没有，就把 http://jetbrains-license-server 复制上去。

### 2. 点击 Activate

### 3. 点击 Test Connection

到此就完成激活了，放一张自己激活后的图。

![激活完成](https://img-blog.csdnimg.cn/20200104145315910.png)