# 磁盘相关的常用命令：
- df -h：查看磁盘占用情况
- df -T：查看所有磁盘的文件系统类型(type)
- fdisk -l：查看所有被系统识别的磁盘
- mount -t type device dir：挂载device到dir

```
[root@pm_operator ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        96G  3.4G   88G   4% /
devtmpfs         32G     0   32G   0% /dev
tmpfs            32G     0   32G   0% /dev/shm
tmpfs            32G  226M   32G   1% /run
tmpfs            32G     0   32G   0% /sys/fs/cgroup
/dev/sda1       477M   65M  383M  15% /boot
/dev/sda2       972G  8.0G  915G   1% /data
tmpfs           6.3G     0  6.3G   0% /run/user/0
```