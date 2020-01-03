# Ansible 组件介绍

本章主要通过对 Ansible 经常使用的组件进行讲解，使对 Ansible 有一个更全面的了解，主要包含以下内容：
1. Ansible Inventory
2. Ansible Ad-Hoc 命令
3. Ansible playbook
4. Ansible facts
5. Ansible role
6. Ansible Galaxy

## Ansible Inventory
Inventory 组件主要存储在配置管理工作中需要管理的不同业务的不同机器的信息。默认 Ansible 的 Inventory 是静态的 INI 格式的文件`/etc/ansible/hosts`，可以通过 ANSIBLE_HOSTS 环境变量指定或者运行 ansible 和 ansible-playbook 的时候用 -i 参数临时设置。

### 定义主机和主机组
首先看下默认 Inventory 文件是如何定义主机和主机组的，默认的 Inventory 文件如下：
```
# - 主机组由[header]元素分隔
# - 您可以输入主机名或IP地址
# - hostname/ip 可以是多个组的成员

# 未组合的主机，在任何主机组之前指定。
## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# 属于'webservers'组的主机集合
## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110
# 如果有多个主机遵循模式
## www[001:006].example.com

# 'dbservers'组中的数据库服务器集合
## [dbservers]
## 
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57
## db-[99:101]-node.example.com
```

### 多个 Inventory 列表
Ansible 支持多个 Inventory 文件，方便管理维护不同业务或环境中的机器。下面介绍如何使用多个 Inventory 文件。

首先新建一个文件夹用来存放 Inventory 文件

`mkdir inventory`

并在文件夹内新建文件，webservers 和 hosts。

hosts 文件如下：
```
10.1.90.59
10.1.90.69
```
webservers 文件如下：
```
[webservers]
10.1.90.59
10.1.90.69
[ansible:children]
webservers
```

然后修改 ansible.cfg 文件中的 inventory 的默认路径

`inventory = /root/ansible/inventory/`

这样就可以使用 ansible 的list-hosts 参数来进行验证

`ansible 10.1.90.59:10.1.90.69 --list-hosts`

返回：
```
  hosts (2):
    10.1.90.59
    10.1.90.69
```

### 动态 Inventory
动态 Inventory 其实可以通过把 ansible.cfg 文件中的 inventory 默认路径改为一个脚本。

> 脚本需要支持两个参数
> - list或者-l ,这个参数显示所有主机以及主机组的信息(json格式)
> - host或者-H ,参数后面指定一个host,会显示这台主机的所有信息(json格式)

下面是 hosts.py 脚本：

```
import argparse
import sys
import json
def list():
    r={}
    h=['10.1.90.'+ str(i) for i in (59,69)]
    hosts={'host':h}
    r['webservers']=hosts
    return  json.dumps(r,indent=4)

def hosts(name):
    r={'ansible_ssh_pass':'123'}
    cpis=dict(r.items())
    return json.dumps(cpis)

if __name__=='__main__':
    parser=argparse.ArgumentParser()
    parser.add_argument('-l','--list',help='host list',action='store_true')
    parser.add_argument('-H','--host',help='hosts vars')
    args=vars(parser.parse_args())
    if args['list']:
        print list()
    elif args['host']:
        print hosts(args['host'])
    else:
        parser.print_help()
```

执行脚本函数 `python hosts.py -l`，返回如下：
```
{
    "webservers": {
        "host": [
            "10.1.90.59", 
            "10.1.90.69"
        ]
    }
}
```

执行脚本函数 `python hosts.py -H 10.1.90.59`，返回如下：`{"ansible_ssh_pass": "123"}`

执行临时指定 hosts.py 脚本，
`ansible -i hosts.py 10.1.90.59 -m ping -o`，返回结果：
`10.1.90.59 | SUCCESS => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": false, "ping": "pong"}`

> 如果报错： [WARNING]:  * Failed to parse /root/ansible/inventory/hosts.py with script plugin: problem running /root/ansible/inventory/hosts.py
--list ([Errno 13] Permission denied)
> 需要给予执行权限
>
> chmod +x hosts.py

> 如果报错："msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"
> 
> 需要安装 sshpass
> 
> yum -y install sshpass

### Inventory 内置参数


参数 | 解释 | 例子
---|---|---
ansible_ssh_host | 定义 host ssh 地址 | ansible_ssh_host=10.1.90.59
ansible_ssh_port | 定义 hosts ssh 端口 | ansible_ssh_port=22
ansible_ssh_user | 定义 hosts ssh 认证用户 | ansible_ssh_user=wupx
ansible_ssh_pass | 定义 hosts ssh 认证密码 | ansible_ssh_pass=123
ansible_sudo | 定义 hosts sudo的用户 | ansible_sudo=wupx
ansible_sudo_pass | 定义 hosts sudo密码 | ansible_sudo_pass=123
ansible_sudo_exe | 定义 hosts sudo 路径 | ansible_sudo_exe=/usr/bin/sudo
ansible_ssh_private_key_file | 定义 hosts 私钥 | ansible_ssh_private_key_file=/root/key
ansible_shell_type | 定义 hosts shell 类型 | ansible_shell_type=bash
ansible_python_interpreter | 定义 hosts 任务执行 python 的路径 | ansible_python_interpreter=/usr/bin/python2.6
ansible_*_interpreter | 定义 hosts 其他语言解析器路径 | ansible_ruby_interpreter=/usr/bin/ruby

## Ansible Ad-Hoc 命令
Ad-Hoc 其实就是临时命令，Ad-Hoc 是相对于 Ansible-playbook 而言的，Ansible 提供两种完成任务方式:一种是 Ad-Hoc 命令集,即ansible，另一种就是 Ansible-playbook，即命令 Ansible-playbook。前者更注重于解决一些简单的或者平时工作中临时遇到的任务，相当于Linux系统命令行下的Shell命令，后者更适合与解决复杂或需固化下来的任务，相当于Linux系统的Shell Scripts。

### 执行命令
Ansible 命令都是并发执行的，默认的并发数由 ansible.cfg 中的 forks 值来确定，也可以在执行命令时通过 -f 指定并发数。

使用命令返回 webservers 组所有主机的 hostname，并指定并发数为 5：`ansible webservers -m shell -a 'hostname' -f 5 -o`

执行结果：
```
192.168.46.129 | CHANGED | rc=0 | (stdout) web2
192.168.46.128 | CHANGED | rc=0 | (stdout) web1
```

使用异步执行，-P 0 的情况下会直接返回 job_id，然后针对主机根据 job_id 查询执行结果：`ansible webservers -B 120 -P 0 -m shell -a 'sleep 10;hostname' -f 5 -o`

执行结果：
```
192.168.46.128 | CHANGED => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "ansible_job_id": "899260515938.13222", "changed": true, "finished": 0, "results_file": "/root/.ansible_async/899260515938.13222", "started": 1}
192.168.46.129 | CHANGED => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "ansible_job_id": "245835357736.13147", "changed": true, "finished": 0, "results_file": "/root/.ansible_async/245835357736.13147", "started": 1}
```

可以根据 job_id 通过 async_status 模块查看异步任务的状态和结果：`ansible 192.168.46.128 -m async_status -a 'jid=899260515938.13222'`

执行结果：
```
192.168.46.128 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "ansible_job_id": "899260515938.13222", 
    "changed": true, 
    "cmd": "sleep 10;hostname", 
    "delta": "0:00:10.024378", 
    "end": "2019-09-22 08:54:35.364400", 
    "finished": 1, 
    "rc": 0, 
    "start": "2019-09-22 08:54:25.340022", 
    "stderr": "", 
    "stderr_lines": [], 
    "stdout": "web1", 
    "stdout_lines": [
        "web1"
    ]
}
```

当-P 参数大于 0 时，Ansible 会自动根据 job_id 轮询查询执行结果：`ansible webservers -B 120 -P 1 -m shell -a 'sleep 10;hostname' -f 5 -o`

执行结果：
```
192.168.46.128 | CHANGED => {"ansible_job_id": "892179643372.13778", "changed": true, "cmd": "sleep 10;hostname", "delta": "0:00:10.023149", "end": "2019-09-22 09:01:57.625917", "finished": 1, "rc": 0, "start": "2019-09-22 09:01:47.602768", "stderr": "", "stderr_lines": [], "stdout": "web1", "stdout_lines": ["web1"]}
192.168.46.129 | CHANGED => {"ansible_job_id": "506204522875.13683", "changed": true, "cmd": "sleep 10;hostname", "delta": "0:00:10.029853", "end": "2019-09-22 09:01:59.039427", "finished": 1, "rc": 0, "start": "2019-09-22 09:01:49.009574", "stderr": "", "stderr_lines": [], "stdout": "web2", "stdout_lines": ["web2"]}
```

### 复制文件
可以使用 copy 模块来批量下发文件，文件的变化是通过 MD5 值来判断的：`ansible webservers -m copy -a 'src=hosts dest=/root/hosts owner=root group=root mode=644 backup=yes' -o`

返回结果：
```
192.168.46.129 | CHANGED => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": true, "checksum": "83e11aa2eb53ae6ea2476c05a8448696018401db", "dest": "/root/hosts", "gid": 0, "group": "root", "md5sum": "a154d2b2131e6420e51e293c540e4cb5", "mode": "0644", "owner": "root", "secontext": "system_u:object_r:admin_home_t:s0", "size": 1021, "src": "/root/.ansible/tmp/ansible-tmp-1569169777.73-89235977935/source", "state": "file", "uid": 0}
192.168.46.128 | CHANGED => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": true, "checksum": "83e11aa2eb53ae6ea2476c05a8448696018401db", "dest": "/root/hosts", "gid": 0, "group": "root", "md5sum": "a154d2b2131e6420e51e293c540e4cb5", "mode": "0644", "owner": "root", "secontext": "system_u:object_r:admin_home_t:s0", "size": 1021, "src": "/root/.ansible/tmp/ansible-tmp-1569169777.73-148086225859296/source", "state": "file", "uid": 0}
```

验证文件下发功能：`ansible webservers -m shell -a 'md5sum /root/hosts' -t 5 -o`

返回结果：
```
192.168.46.128 | CHANGED | rc=0 | (stdout) a154d2b2131e6420e51e293c540e4cb5  /root/hosts
192.168.46.129 | CHANGED | rc=0 | (stdout) a154d2b2131e6420e51e293c540e4cb5  /root/hosts
```

### 包和服务管理
可以直接使用 Ad-Hoc 命令来管理包和服务：
```
ansible webservers -m yum -a 'name=httpd state=latest' -f 5 -o

ansible webservers -m service -a 'name=httpd state=started' -f 5 -o

ansible webservers -m shell -a 'rpm -qa httpd' -f 5 -o
 ```

验证服务运行情况：`ansible webservers -m shell -a 'netstat -tpln|grep httpd' -f 5`

返回结果：
```
192.168.46.128 | CHANGED | rc=0 >>
tcp6       0      0 :::80                   :::*                    LISTEN      16853/httpd         

192.168.46.129 | CHANGED | rc=0 >>
tcp6       0      0 :::80                   :::*                    LISTEN      43184/httpd
```
### 用户管理
首先通过 openssl 命令生成密码（因为ansible user 的 password 参数需要接受加密后的值）：`echo ansible | openssl passwd -1 -stdin`

返回结果：`$1$RBXBgM3M$WE3mYCc2gIlFIircO3unx.`

使用 user 模块批量新建用户：`ansible webservers -m user -a 'name=test password="$1$RBXBgM3M$WE3mYCc2gIlFIircO3unx."' -f 5 -o`

返回结果：
```
192.168.46.128 | CHANGED => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": true, "comment": "", "create_home": true, "group": 1001, "home": "/home/test", "name": "test", "password": "NOT_LOGGING_PASSWORD", "shell": "/bin/bash", "state": "present", "system": false, "uid": 1001}
192.168.46.129 | CHANGED => {"ansible_facts": {"discovered_interpreter_python": "/usr/bin/python"}, "changed": true, "comment": "", "create_home": true, "group": 1001, "home": "/home/test", "name": "test", "password": "NOT_LOGGING_PASSWORD", "shell": "/bin/bash", "state": "present", "system": false, "uid": 1001}
```

通过 SSH 登录验证新建用户是否成功（密码为 ansible）：`ssh 192.168.46.128 -l test`

## Ansible playbook
playbook 是 Ansible 进行配置管理的组件，是来弥补 Ad-Hoc 命令无法支撑复杂环境的配置管理工作的。playbook 是 Ansible 的重要组件之一，因此放在下一篇来对 Ansible 的 playbook 进行详细讲解。

## Ansible facts
facts 组件是 Ansible 用于采集被管机器设备信息的功能，可使用 setup 模块查看机器所有 facts 信息，或使用 filter 来查看指定信息（返回的结果是 JSON 格式）。

查看机器的所有 facts 信息：`ansible 192.168.46.128 -m setup`

查看机器的 ipv4 信息：`ansible 192.168.46.128 -m setup -a 'filter=ansible_all_ipv4_addresses'`

返回结果：
```
192.168.46.128 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.46.128"
        ], 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
```

### 使用 facter 扩展 facts 信息
Ansible 的 facts 组件会判断被控机器上是否安装 facter 和 ruby-json 包，若存在，Ansible 的 facts 会采集 facter 信息。

查看是否安装 facter 和 ruby-json：`ansible 192.168.46.128 -m shell -a 'rpm -qa ruby-json facter'`

运行 facter 模块查看 facter 信息：`ansible 192.168.46.128 -m facter`

### 使用 ohai 扩展 facts 信息
Ansible 的 facts 组件会判断被控机器上是否安装 ohai 包，若存在，Ansible 的 facts 会采集 ohai 信息。

查看是否安装 ohai：`ansible 192.168.46.128 -m shell -a 'gem list|grep ohai'`

运行 ohai 模块查看 ohai 信息：`ansible 192.168.46.128 -m ohai`

> 直接运行 setup 模块也会采集 facter 和 ohai 信息。

## Ansible role
role 只是对我们使用的 playbook 的目录结构进行一些规范。

这是一个 role 的目录结构：

```
├── roles
    ├── dbsrvs -------------role1名称
    │   ├── files -------------ansible中unarchive、copy等模块会自动来这里找文件，从而我们不必写绝对路径，只需写文件名
    │   │   ├── mysql.tar.gz
    │   │   └── nginx.tar.gz
    │   ├── handlers -----------存放tasks中的notify指定的内容
    │   │   └── main.yml
    │   ├── meta
    │   ├── tasks --------------存放playbook的目录，其中main.yml是主入口文件，在main.yml中导入其他yml文件，要采用import_tasks关键字，include要弃用了
    │   │   ├── install.yml
    │   │   └── main.yml -------主入口文件
    │   ├── templates ----------存放模板文件。template模块会将模板文件中的变量替换为实际值，然后覆盖到客户机指定路径上
    │   │   └── nginx.conf.j2
    │   └── vars ----------存放变量文件
    └── websrvs -------------role2名称
    │   ├── files
    │   │   ├── mysql.tar.gz
    │   │   └── nginx.tar.gz
    │   ├── handlers
    │   │   └── main.yml
    │   ├── meta
    │   ├── tasks
    │   │   ├── install.yml
    │   │   └── main.yml
    │   ├── templates
    │   │   └── nginx.conf.j2
    │   └── vars
    └── site.yml -------------role引用的入口文件
```

执行 role：`ansible-playbook -i /etc/ansible/hosts site.yml`

## Ansible Galaxy
[Galaxy](https://galaxy.ansible.com) 是 Ansible 官方分享 role 的功能平台。可将自己编写的 role 通过 ansible-galaxy 上传到 Galaxy 网站。也可通过 ansible-galaxy 命令实现 role 的分享和安装。使用`ansible-galaxy install`就可以安装 role，默认安装路径为`/etc/ansible/roles/`。

## 总结
本章主要介绍一些 Ansible 常用的组件，主要包含 Ansible Inventory、playbook、facts、role、Galaxy 等。下一篇将对 Ansible 中的 playbook 进行详细讲解。