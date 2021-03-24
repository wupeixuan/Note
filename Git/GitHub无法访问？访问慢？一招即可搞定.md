GitHub 是一个面向开源及私有软件项目的托管平台，程序员可以在上面探索感兴趣的项目，几乎是程序员的圣地。

最近听群里朋友说 GitHub 无法访问或者访问速度极其慢，经常打开 GitHub 变成这样：

![](https://wupx-1256189981.file.myqcloud.com/img/202103/23/1616487915.png)

这怎么能行呢？无疑妨碍了我们的学习之路呀。

下面我就来把我的解决方法和大家说下：

## 下载 SwitchHosts

首先下载 SwitchHosts，可以更方便的切换 hosts，下载地址：`https://github.com/oldj/SwitchHosts/releases`

考虑到 GitHub 访问不稳定，可以公众号【武培轩】回复【hosts】获取软件安装包。

下面就需要用到 GitHub 上的一个项目：[GitHub520](https://github.com/521xueweihan/GitHub520)，他可以定时提供最新的 hosts 配置来使访问 GitHub 更加顺畅。

## 添加 hosts 规则

接下来打开 SwitchHosts，然后添加 hosts 规则：

- 方案名（Title）随便写
- 类型（Type）选择远程
- URL 地址栏输入 `https://cdn.jsdelivr.net/gh/521xueweihan/GitHub520@main/hosts`
- 自动更新建议选择 1 小时一更新

![添加hosts规则](https://wupx-1256189981.file.myqcloud.com/img/202103/23/1616489473.png)

## 其他方法

若不想下载 SwitchHosts 等类似软件，可以手动修改 hosts 文件，针对各个系统的文件位置是不同的，具体位置如下：

- Windows 系统：`C:\Windows\System32\drivers\etc\hosts`
- Mac 系统：`/etc/hosts`

在 hosts 文件中添加如下内容：

```
185.199.108.154               github.githubassets.com
140.82.114.21                 central.github.com
185.199.108.133               desktop.githubusercontent.com
185.199.108.153               assets-cdn.github.com
185.199.108.133               camo.githubusercontent.com
185.199.108.133               github.map.fastly.net
199.232.69.194                github.global.ssl.fastly.net
140.82.112.3                  gist.github.com
185.199.108.153               github.io
140.82.112.3                  github.com
140.82.114.5                  api.github.com
185.199.108.133               raw.githubusercontent.com
185.199.108.133               user-images.githubusercontent.com
185.199.108.133               favicons.githubusercontent.com
185.199.108.133               avatars5.githubusercontent.com
185.199.108.133               avatars4.githubusercontent.com
185.199.108.133               avatars3.githubusercontent.com
185.199.108.133               avatars2.githubusercontent.com
185.199.108.133               avatars1.githubusercontent.com
185.199.108.133               avatars0.githubusercontent.com
185.199.108.133               avatars.githubusercontent.com
140.82.113.9                  codeload.github.com
52.216.187.11                 github-cloud.s3.amazonaws.com
52.216.162.99                 github-com.s3.amazonaws.com
52.216.142.196                github-production-release-asset-2e65be.s3.amazonaws.com
52.217.97.236                 github-production-user-asset-6210df.s3.amazonaws.com
52.217.194.41                 github-production-repository-file-5c1aeb.s3.amazonaws.com
185.199.108.153               githubstatus.com
64.71.168.201                 github.community
185.199.108.133               media.githubusercontent.com
```

如果已经失效，可以公众号【武培轩】回复【hosts】获取最新的 hosts。

### 激活生效

大部分情况下是直接生效，如未生效可尝试下面的办法，刷新 DNS：

1. Windows：在 CMD 窗口输入：`ipconfig /flushdns`
2. Linux 命令：`sudo rcnscd restart`
3. Mac 命令：`sudo killall -HUP mDNSResponder`

相信大家通过上述方法配置后，就可以顺畅地访问 GitHub 了！