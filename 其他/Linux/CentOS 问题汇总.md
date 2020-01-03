# 问题描述
`-bash: ./vmware-install.pl: /usr/bin/perl: bad interpreter: No such file or directory `

# 解决办法
安装相关环境
`yum -y install gcc gcc-c++ perl make kernel-headers kernel-devel` 

# 问题描述
CentOS 7 下 ifconfig command not found 解决办法

# 解决办法
1.查看ifconfig命令是否存在

　　查看 /sbin/ifconfig是否存在

 

2.如果ifconfig命令存在，查看环境变量设置  

　　[root@localhost ~]# echo $PATH

　　如果环境变量中没有包含ifconfig命令的路径,修改PATH变量使之包含/sbin路径：

      打开/etc/profile文件，在文件最好一行加上 export PATH=$PATH:/sbin 保存并重启即可。

 

3.如果ifconfig命令不存在

　　[root@localhost ~]# yum upgrade

　　[root@localhost ~]# yum install net-tools