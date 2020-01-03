# Ansible 安装与配置

本章主要讲的是 Ansible 安装与基本配置，主要包含以下内容：
1. Ansible 环境准备
2. 安装 Ansible
3. 配置运行环境
4. Ansible实践

## Ansible 环境准备
从 GitHub 获取 Ansible，准备控制主机，查看被管节点。

使用的操作系统为 Centos 7.0，自带 Python 2.7.5。

角色 | 主机名 | IP 地址 | 组名 | CPU | Web 根目录 
---|---|---|---|---|---
被管节点 | web1 | 192.168.46.128 | webservers | 2 | /website
被管节点 | web2 | 192.168.46.129 | webservers | 2 | /website
控制节点 | ansiblecontrol | 192.168.46.130 | --- | --- | ---

> 永久性的修改主机名称`hostnamectl set-hostname web1`

## 安装 Ansible
Ansible 的安装方式分为直接用源码安装以及用包管理工具安装。
### 直接用源码安装
从 GitHub 源码库安装方式

1. 提取 Ansible 源代码
    ```sh
    git clone https://github.com/ansible/ansible.git --recursive
    cd ./ansible
    # 减少告警/错误信息输出，可在安装时加上 -q 参数
    source ./hacking/env-setup -q
    ```
2. 若没有安装 pip，安装对应 Python 版本的 pip
    ```
    sudo easy_install pip
    ```
3. 安装 Ansible 控制主机需要的 Python 模块
    ```
    sudo pip install paramiko PyYAML Jinja2 httplib2 six
    ```
4. 当更新 Ansible 版本时，要更新 git 源码树以及 git 中指向 Ansible 自身的模块（称为 submodules）
    ```
    git pull --rebase
    git submodule update --init --recursive
    ```
5. 运行 env-setup 脚本（默认资源清单 inventory 文件是 /etc/ansible/hosts）
    ```
    .. code-block:: bash
    echo "127.0.0.1" > ~/ansible_hosts
    export ANSIBLE_HOSTS=~/ansible_hosts
    ```
> 通过 GitHub 仓库安装的，需要把仓库中 examples 目录下的 ansible.cfg 复制到 /etc/ansible 目录下

### 用包管理工具安装
pip安装方式

```
#安装 pip
sudo easy_install pip
#通过 pip 命令安装 Ansible
sudo pip install ansible
```
> 通过 pip 安装的，没有自动生成的配置文件，需要自己新建 /etc/ansible/ansible.cfg

## 配置运行环境
配置文件优先级：
1. ANSIBLE_CONFIG：首先，Ansible 命令会检查环境变量，以及环境变量指向的配置文件。
2. ./ansible.cfg：其次，会检查当前目录下的 ansible.cfg 配置文件。
3. ~/ansible.cfg：再次，会检查当前用户 home 目录下的 ansible.cfg 配置文件。
4. /etc/ansible/ansible.cfg：最后，会检查安装时自动生成的配置文件。

### 配置 Ansible 环境

1. 使用环境变量方式配置
2. 设置 ansible.cfg 配置参数

```
[defaults]

#inventory      = /etc/ansible/hosts    #inventory文件路径
#library        = /usr/share/my_modules/    #模块文件路径
#module_utils   = /usr/share/my_module_utils/   #自定义模块工具存放目录
#remote_tmp     = ~/.ansible/tmp    #临时文件远程主机存放目录
#local_tmp      = ~/.ansible/tmp    #临时文件本地存放目录
#plugin_filters_cfg = /etc/ansible/plugin_filters.yml
#forks          = 5 #默认开启的进程数
#poll_interval  = 15    #默认轮询时间间隔
#sudo_user      = root  #默认sudo用户
#ask_sudo_pass = True   #是否需要sudo密码
#ask_pass      = True   #是否需要密码
#transport      = smart 通信机制，如果本地系统支持 ControlPersist技术的话,将会使用(基于OpenSSH)‘ssh’,如果不支持将使用‘paramiko’，其他传输选项‘local’,‘chroot’,’jail’等等
#remote_port    = 22    #连接被管节点的管理端口
#module_lang    = C #模块运行的语言环境
#module_set_locale = False
#gathering = implicit   #facts信息收集开关，implicit（默认不收集）
#gather_subset = all    #facts 的收集范围
# gather_timeout = 10   #收集超时间隔

# Ansible facts are available inside the ansible_facts.* dictionary
# namespace. This setting maintains the behaviour which was the default prior
# to 2.5, duplicating these variables into the main namespace, each with a
# prefix of 'ansible_'.
# This variable is set to True by default for backwards compatibility. It
# will be changed to a default of 'False' in a future release.
# ansible_facts.
# inject_facts_as_vars = True

#roles_path    = /etc/ansible/roles #role存放路径

#host_key_checking = False  #是否检查SSH主机的密钥

# change the default callback, you can only have one 'stdout' type  enabled at a time.
#stdout_callback = skippy

# enable callback plugins, they can output to stdout but cannot be 'stdout' type.
#callback_whitelist = timer, mail

# Determine whether includes in tasks and handlers are "static" by
# default. As of 2.0, includes are dynamic by default. Setting these
# values to True will make includes behave more like they did in the
# 1.x versions.
#task_includes_static = False
#handler_includes_static = False

# Controls if a missing handler for a notification event is an error or a warning
#error_on_missing_handler = True

#sudo_exe = sudo    #ansible sudo执行命令
#sudo_flags = -H -S -n  #ansible sudo执行参数
#timeout = 10   #ansible SSH连接的超时间隔/秒
#remote_user = root #ansible 远程认证用户
#log_path = /var/log/ansible.log    #指定存储日志的文件
#module_name = command  #ansible 默认执行模块

#executable = /bin/sh   #ansible 命令执行 shell

# if inventory variables overlap, does the higher precedence one win
# or are hash values merged together?  The default is 'replace' but
# this can also be set to 'merge'.
#hash_behaviour = replace   #ansible 主机变量重复处理方式

# by default, variables from roles will be visible in the global variable
# scope. To prevent this, the following option can be enabled, and only
# tasks and handlers within the role will see the variables there
#private_role_vars = yes

#jinja2_extensions = jinja2.ext.do,jinja2.ext.i18n  #Jinja2 扩展列表

#private_key_file = /path/to/file   #ansible ssh 私钥文件

# If set, configures the path to the Vault password file as an alternative to
# specifying --vault-password-file on the command line.
#vault_password_file = /path/to/vault_password_file


#ansible_managed = Ansible managed: {file} modified on %Y-%m-%d %H:%M:%S by {uid} on {host} #在 jinja2 中格式化 ansible_managed 变量
#ansible_managed = Ansible managed

#display_skipped_hosts = True   #开启显示跳过的主机

# by default, if a task in a playbook does not include a name: field then
# ansible-playbook will construct a header that includes the task's action but
# not the task's args.  This is a security feature because ansible cannot know
# if the *module* considers an argument to be no_log at the time that the
# header is printed.  If your environment doesn't have a problem securing
# stdout from ansible-playbook (or you have manually specified no_log in your
# playbook on all of the tasks where you have secret information) then you can
# safely set this to True to get more informative messages.
#display_args_to_stdout = False

#error_on_undefined_vars = False    #开启错误，或者没有定义的变量

#system_warnings = True #开启第三方包系统警告

#deprecation_warnings = True    #配置是否显示弃用警告

# (as of 1.8), Ansible can optionally warn when usage of the shell and
# command module appear to be simplified by using a default Ansible module
# instead.  These warnings can be silenced by adjusting the following
# setting or adding warn=yes or warn=no to the end of the command line
# parameter string.  This will for example suggest using the git module
# instead of shelling out to the git command.
# command_warnings = False


# set plugin path directories here, separate with colons
#action_plugins     = /usr/share/ansible/plugins/action #ansible action 插件路径
#become_plugins     = /usr/share/ansible/plugins/become
#cache_plugins      = /usr/share/ansible/plugins/cache
#callback_plugins   = /usr/share/ansible/plugins/callback
#connection_plugins = /usr/share/ansible/plugins/connection
#lookup_plugins     = /usr/share/ansible/plugins/lookup
#inventory_plugins  = /usr/share/ansible/plugins/inventory
#vars_plugins       = /usr/share/ansible/plugins/vars
#filter_plugins     = /usr/share/ansible/plugins/filter
#test_plugins       = /usr/share/ansible/plugins/test
#terminal_plugins   = /usr/share/ansible/plugins/terminal
#strategy_plugins   = /usr/share/ansible/plugins/strategy


# by default, ansible will use the 'linear' strategy but you may want to try
# another one
#strategy = free

#bin_ansible_callbacks = False  #开启 ansible 命令加载 callback 插件

#nocows = 1 #是否开启 ansiblenocows 图形

# set which cowsay stencil you'd like to use by default. When set to 'random',
# a random stencil will be selected for each task. The selection will be filtered
# against the `cow_whitelist` option below.
#cow_selection = default
#cow_selection = random

# when using the 'random' option for cowsay, stencils will be restricted to this list.
# it should be formatted as a comma-separated list with no spaces between names.
# NOTE: line continuations here are for formatting purposes only, as the INI parser
#       in python does not support them.
#cow_whitelist=bud-frogs,bunny,cheese,daemon,default,dragon,elephant-in-snake,elephant,eyes,\
#              hellokitty,kitty,luke-koala,meow,milk,moofasa,moose,ren,sheep,small,stegosaurus,\
#              stimpy,supermilker,three-eyes,turkey,turtle,tux,udder,vader-koala,vader,www

#nocolor = 1    #是否开启 ansible 颜色输出

# if set to a persistent type (not 'memory', for example 'redis') fact values
# from previous runs in Ansible will be stored.  This may be useful when
# wanting to use, for example, IP information from one group of servers
# without having to talk to them in the same playbook run to get their
# current IP information.
#fact_caching = memory

#This option tells Ansible where to cache facts. The value is plugin dependent.
#For the jsonfile plugin, it should be a path to a local directory.
#For the redis plugin, the value is a host:port:database triplet: fact_caching_connection = localhost:6379:0

#fact_caching_connection=/tmp



# retry files
# When a playbook fails a .retry file can be created that will be placed in ~/
# You can enable this feature by setting retry_files_enabled to True
# and you can change the location of the files by setting retry_files_save_path

#retry_files_enabled = False
#retry_files_save_path = ~/.ansible-retry

# squash actions
# Ansible can optimise actions that call modules with list parameters
# when looping. Instead of calling the module once per with_ item, the
# module is called once with all items at once. Currently this only works
# under limited circumstances, and only with parameters named 'name'.
#squash_actions = apk,apt,dnf,homebrew,pacman,pkgng,yum,zypper

# prevents logging of task data, off by default
#no_log = False

# prevents logging of tasks, but only on the targets, data is still logged on the master/controller
#no_target_syslog = False

# controls whether Ansible will raise an error or warning if a task has no
# choice but to create world readable temporary files to execute a module on
# the remote machine.  This option is False by default for security.  Users may
# turn this on to have behaviour more like Ansible prior to 2.1.x.  See
# https://docs.ansible.com/ansible/become.html#becoming-an-unprivileged-user
# for more secure ways to fix this than enabling this option.
#allow_world_readable_tmpfiles = False

# controls the compression level of variables sent to
# worker processes. At the default of 0, no compression
# is used. This value must be an integer from 0 to 9.
#var_compression_level = 9

# controls what compression method is used for new-style ansible modules when
# they are sent to the remote system.  The compression types depend on having
# support compiled into both the controller's python and the client's python.
# The names should match with the python Zipfile compression types:
# * ZIP_STORED (no compression. available everywhere)
# * ZIP_DEFLATED (uses zlib, the default)
# These values may be set per host via the ansible_module_compression inventory
# variable
#module_compression = 'ZIP_DEFLATED'

#max_diff_size = 1048576    #diff文件的大小限制/字节

# This controls how ansible handles multiple --tags and --skip-tags arguments
# on the CLI.  If this is True then multiple arguments are merged together.  If
# it is False, then the last specified argument is used and the others are ignored.
# This option will be removed in 2.8.
#merge_multiple_cli_flags = True

# Controls showing custom stats at the end, off by default
#show_custom_stats = True

# Controls which files to ignore when using a directory as inventory with
# possibly multiple sources (both static and dynamic)
#inventory_ignore_extensions = ~, .orig, .bak, .ini, .cfg, .retry, .pyc, .pyo

# This family of modules use an alternative execution path optimized for network appliances
# only update this setting if you know how this works, otherwise it can break module execution
#network_group_modules=eos, nxos, ios, iosxr, junos, vyos

# When enabled, this option allows lookups (via variables like {{lookup('foo')}} or when used as
# a loop with `with_foo`) to return data that is not marked "unsafe". This means the data may contain
# jinja2 templating language which will be run through the templating engine.
# ENABLING THIS COULD BE A SECURITY RISK
#allow_unsafe_lookups = False

# set default errors for all plays
#any_errors_fatal = False

[inventory]
# enable inventory plugins, default: 'host_list', 'script', 'auto', 'yaml', 'ini', 'toml'
#enable_plugins = host_list, virtualbox, yaml, constructed

# ignore these extensions when parsing a directory as inventory source
#ignore_extensions = .pyc, .pyo, .swp, .bak, ~, .rpm, .md, .txt, ~, .orig, .ini, .cfg, .retry

# ignore files matching these patterns when parsing a directory as inventory source
#ignore_patterns=

# If 'true' unparsed inventory sources become fatal errors, they are warnings otherwise.
#unparsed_is_failed=False

[privilege_escalation]
#become=True    #是否开启 become 模式
#become_method=sudo #定义 become 方式
#become_user=root   #定义 become 用户
#become_ask_pass=False  #是否定义 become 提示密码

[paramiko_connection]

#record_host_keys=False #是否记录主机 key

#pty=False  #是否开启命令执行伪终端

# paramiko will default to looking for SSH keys initially when trying to
# authenticate to remote devices.  This is a problem for some network devices
# that close the connection after a key failure.  Uncomment this line to
# disable the Paramiko look for keys function
#look_for_keys = False

# When using persistent connections with Paramiko, the connection runs in a
# background process.  If the host doesn't already have a valid SSH key, by
# default Ansible will prompt to add the host key.  This will cause connections
# running in background processes to fail.  Uncomment this line to have
# Paramiko automatically add host keys.
#host_key_auto_add = True

[ssh_connection]
#SSH 连接配置
#ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s  #ansib ssh参数，ControlMaster用于设置是否启用 SSH的Multiplexing，关闭则写no，ControlPersist为SSH session保持的时间

# control_path_dir = /tmp/.ansible/cp   #ansible ssh 长连接控制文件
#control_path_dir = ~/.ansible/cp

# The path to use for the ControlPath sockets. This defaults to a hashed string of the hostname,
# port and username (empty string in the config). The hash mitigates a common problem users
# found with long hostnames and the conventional %(directory)s/ansible-ssh-%%h-%%p-%%r format.
# In those cases, a "too long for Unix domain socket" ssh error would occur.
#
# Example:
# control_path = %(directory)s/%%h-%%r
#control_path =

#pipelining = False #是否开启 pipelining 模式

#scp_if_ssh = smart #是否开启scp模式推送脚本，smart（先尝试sftp推送，再尝试scp推送）

# Control the mechanism for transferring files (new)
# If set, this will override the scp_if_ssh option
#   * sftp  = use sftp to transfer files
#   * scp   = use scp to transfer files
#   * piped = use 'dd' over SSH to transfer files
#   * smart = try sftp, scp, and piped, in that order [default]
#transfer_method = smart

# if False, sftp will not use batch mode to transfer files. This may cause some
# types of file transfer failures impossible to catch however, and should
# only be disabled if your sftp version has problems with batch mode
#sftp_batch_mode = False

# The -tt argument is passed to ssh when pipelining is not enabled because sudo 
# requires a tty by default. 
#usetty = True

#retries = 3    #重试与主机SSH连接的次数

[persistent_connection]
#持久连接配置
#connect_timeout = 30   #持久连接超时间隔

# The command timeout value defines the amount of time to wait for a command
# or RPC call before timing out. The value for the command timeout must
# be less than the value of the persistent connection idle timeout (connect_timeout)
# The default value is 30 second.
#command_timeout = 30

[accelerate]
#accelerate_port = 5099 #accelerate 远程监听端口
#accelerate_timeout = 30    #accelerate 模式，命令执行超时时间/秒
#accelerate_connect_timeout = 5.0   #accelerate 模式，连接超时时间/秒

# The daemon timeout is measured in minutes. This time is measured
# from the last activity to the accelerate daemon.
#accelerate_daemon_timeout = 30

# If set to yes, accelerate_multi_key will allow multiple
# private keys to be uploaded to it, though each user must
# have access to the system via SSH to add a new key. The default
# is "no".
#accelerate_multi_key = yes

[selinux]
#上下文配置
#special_context_filesystems=nfs,vboxsf,fuse,ramfs,9p,vfat
#libvirt_lxc_noseclabel = yes

[colors]
#highlight = white
#verbose = blue
#warn = bright purple
#error = red
#debug = dark gray
#deprecate = purple
#skip = cyan
#unreachable = red
#ok = green
#changed = yellow
#diff_add = green
#diff_remove = red
#diff_lines = cyan


[diff]
# always = no   #是否一直打印diff
# context = 3   #diff中显示的上下文行数
```

### 配置 Linux 主机 SSH 无密码访问
为避免 Ansible 下发指令时需要输入目标主机密码，通过证书签名达到 SSH 无密码访问。使用 ssh-keygen 和 ssh-copy-id 来实现快速证书的生成及公钥的下发。

在控制主机上创建密钥，执行`ssh-keygen -t rsa`，将在 /root/.ssh/ 下生成密钥，其中 id_rsa 为私钥， id_rsa.pub 为公钥。
```
#生成密钥
ssh-keygen  -t rsa
```

下发密钥就是控制主机将公钥 is_rsa.pub 下发到被管节点上用户下的 .ssh 目录，并重命名为 authorized_keys，且权限值为400。
```
#下发公钥到 web1（192.168.46.128）
ssh-copy-id -i id_rsa root@192.168.46.128
#ssh连接验证
ssh root@192.168.46.128
#退出
exit
```

## Ansible 实践

### 主机连通性测试
修改主机与组配置 /etc/ansible/hosts ，添加两台主机的ip地址，同时定义一个 webservers 组包含这两个地址

```
192.168.46.128
192.168.46.129

[webservers]
192.168.46.128
192.168.46.129
```

用 ping 模块对单台主机进行 ping 操作
`ansible 192.168.46.128 -m ping`

结果如下
```
192.168.46.128 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```

对 webservers 组进行 ping 操作
`ansible webservers -m ping`

> 在命令后加 -v 或 -vvv 可得到详细的输出结果

结果如下
```
192.168.46.128 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
192.168.46.129 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```

### 在被管节点上批量执行命令

在 home 目录下创建资源清单文件 inventory.cfg

`vim inventory.cfg`

内容如下：
```
[webservers]
192.168.46.128
192.168.46.129
```

用 Ansible 的 shell 模块 在 webservers 组的服务器上显示 hello ansible（用 common 模块也可以实现）

`ansible webservers -m shell -a '/bin/echo hello ansible' -i inventory.cfg `

结果如下：
```
192.168.46.128 | CHANGED | rc=0 >>
hello ansible

192.168.46.129 | CHANGED | rc=0 >>
hello ansible
```

### Ansible 获取帮助信息

`ansible-doc -h` 获得帮助

`ansible-doc -l` 获得工具下可使用的模块

`ansible-doc -s` 获得工具下模块支持的动作


## 总结
通过在 CentOS 上以不同的方式安装 Ansible 以及对 Ansible 进行参数配置，并通过 Ansible 在被管节点上执行命令。
