Notepad ++是一个十分强大的编辑器，除了可以用来制作一般的纯文字说明文件，也十分适合编写计算机程序代码。Notepad ++不仅有语法高亮度显示，也有语法折叠功能，并且支持宏以及扩充基本功能的外挂模组。但是对Markdown支持不够。

这里通过插件与自定义语法让Notepad++变成一个Markdown书写工具。

## Markdown语法高亮
### 下载所需文件
因为通过GitHub下载一直超时，我就直接打包放在博客园了。

[下载链接](https://files.cnblogs.com/files/wupeixuan/notepad_markdown.zip)

### 导入语法规则
打开Notepad++，点击“语言” ，选择“自定义语言格式” ，点击“导入”，选择下载并解压后文件夹中的“userDefineLang_markdown.xml”文件。

导入完成后重启notepad++，点击“语言”，选择“Markdown”即可。

![设置](http://images.cnblogs.com/cnblogs_com/wupeixuan/1188893/o_%e5%9b%be%e7%89%871_%e7%9c%8b%e5%9b%be%e7%8e%8b.png)

## Markdown实时预览
### 安装实时预览插件
打开Notepad++，点击“设置”，选择“导入-导入插件”，将之前下载的文件中的“NppMarkdown.dll”导入即可。
### 打开插件
打开Notepad++，点击“插件”，选择“NppMarkdown” 在右侧出现的“preview markdown”窗口底部，勾选“live preview” 同时点击“preview”即可。

![设置](http://images.cnblogs.com/cnblogs_com/wupeixuan/1188893/o_%e5%be%ae%e4%bf%a1%e6%88%aa%e5%9b%be_20180329234049.png)

