Maven工具是强大的项目管理工具，有效的使用Maven将大大提升代码的生产效率。

## 安装Maven
进入 https://maven.apache.org/download.cgi 网站，因为我是windows系统，下载的是bin.zip，下载完成后解压到指定目录，解压后文档结构如下所示：

![image](http://images.cnblogs.com/cnblogs_com/wupeixuan/1207112/o_%e5%be%ae%e4%bf%a1%e5%9b%be%e7%89%87_20180427150929.png)

--bin：保存Maven的可执行命令，mvn和mvn.bat就是执行Maven工具的命令。

--boot：该目录只包含一个plexus-classworlds-2.5.2.jar，plexus-classworlds-2.5.2.jar是一个类加载框架。

--conf：保存Maven配置文件的目录，该目录包含setting.xml文件，该文件用于设置Maven的全局行为。

--lib：该目录包含了所有Maven运行时需要的类库，此外，还包含Maven所依赖的第三方类库。

--LICENSE、README.txt等说明文档。

## 配置Maven
Maven需要配置如下两个环境变量：

JAVA_HOME：该环境变量指向JDK安装路径。

MAVEN_HOME：该环境变量指向MAVEN安装路径。

最好添加PATH环境变量中：PATH：;%MAVEN_HOME%\bin;

## 检查
安装配置完成后，在命令行执行mvn -v命令，会输出如下结果：
```
C:\Users\wpx>mvn -v
Apache Maven 3.5.3 (3383c37e1f9e9b3bc3df5050c29c8aff9f295297; 2018-02-25T03:49:05+08:00)
Maven home: D:\SoftWare\apache-maven-3.5.3\bin\..
Java version: 1.8.0_151, vendor: Oracle Corporation
Java home: D:\SoftWare\jdk1.8.0_151\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```
如果你看到类似结果，说明 Maven 在 Windows 上已安装配置成功。

## 在IDEA中配置Maven
打开 File-Settings，更改Maven的配置
![image](http://images.cnblogs.com/cnblogs_com/wupeixuan/1207112/o_%e5%be%ae%e4%bf%a1%e6%88%aa%e5%9b%be_20180427155536.png)


