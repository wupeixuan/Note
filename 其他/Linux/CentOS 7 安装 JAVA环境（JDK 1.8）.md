## 卸载CentOS默认安装的OpenJDK
查看是否安装 OpenJDK

`java -version`


```
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (build 1.8.0_222-b10)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)

```

查看安装位置

`rpm -qa | grep java`

```
javamail-1.4.6-8.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
tzdata-java-2019b-1.el7.noarch
javapackages-tools-3.4.1-11.el7.noarch
java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_64
java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
javassist-3.16.1-10.el7.noarch
java-1.8.0-openjdk-devel-1.8.0.222.b10-0.el7_6.x86_64

```

执行语句删除openjdk

```
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.222.b10-0.el7_6.x86_64
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.222.b10-0.el7_6.x86_64
rpm -e --nodeps java-1.8.0-openjdk-devel-1.8.0.222.b10-0.el7_6.x86_64
```
检查是否删除

`java -version`

```
-bash: /usr/bin/java: No such file or directory
```

## 安装Oracle Java JDK 8
从官网下载jdk-8u221-linux-x64.tar.gz。

下载后通过ftp上传到服务器。

创建目录，解压

```
mkdir /usr/java
tar zvxf jdk-8u221-linux-x64.tar.gz -C /usr/java
```

环境配置，修改profile文件

`vi /etc/profile`

添加

```
export JAVA_HOME=/usr/java/jdk1.8.0_221
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

使环境变量生效

`source /etc/profile`

检查是否配置成功

`java -version`

```
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

