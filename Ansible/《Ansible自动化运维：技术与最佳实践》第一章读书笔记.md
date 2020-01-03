# Ansible 架构及特点

本章主要讲的是 Ansible 架构及特点，主要包含以下内容：
1. Ansible 软件
2. Ansible 架构模式
3. Ansible 特性

## Ansible 软件
Ansible 的编排引擎可以完成配置管理、流程控制、资源部署等工作。 Ansible 基于 Python语言实现，由 Paramiko 和 PyYAML 两个关键模块构建。

### Ansible 应用领域
1. 配置管理
2. 服务即时开通
3. 应用部署
4. 流程编排
5. 监控告警
6. 日志记录

## Ansible 架构模式
Ansible 维护模式通常由控制机和被管机组成。控制机是用来安装 Ansible 工具软件、执行维护指令的服务器或工作站，是 Ansible 维护的核心。被管机是运行业务服务的服务器，由控制机通过SSH来进行管理。

### Ansible 管理方式
Ansible 是一个模型驱动的配置管理器，支持多节点发布、远程任务执行。默认使用SSH进行远程连接。无需再被管节点上安装附加软件，可使用各种编程语言进行扩展。

Ansible 管理系统由控制主机和一组被管节点组成。控制主机直接通过SSH控制被管节点，被管节点通过 Ansible 的资源清单来进行分组管理。
!["Ansible 总体系统构成"](https://img2018.cnblogs.com/blog/1356806/201909/1356806-20190908125059321-45390010.jpg  "Ansible 总体系统构成")


#### Ansible 用剧本方式对3台运行 Nginx 服务的 Ubuntu 服务器进行配置管理
编写 webservers.yml 的 Ansible 脚本，即 playbook ，其中包含被管节点的 hosts 和对这些 hosts 按照顺序执行的任务列表（task）。

hosts 包括web1、web2、web3。

任务列表包括如下过程：
- 安装 Nginx（Install Nginx）
- 创建 Nginx 配置文件（/etc/nginx/nginx.conf）
- 基于安全证书SSH方式拷贝配置文件，重启 Nginx 服务
- 确保 Nginx 服务处于启动状态

在 Ansible 系统的控制主机上执行`ansible-playbook webservers.yml`，Ansible 将会通过 SSH 连接并行地在web1、web2、web3上面安装、配置、运行 Nginx 服务。

!["运行 Ansible 的 playbook 对 3 台 Web 服务器进行配置"](https://img2018.cnblogs.com/blog/1356806/201909/1356806-20190908123603936-901913120.png "运行 Ansible 的 playbook 对 3 台 Web 服务器进行配置")


### Ansible 系统架构
!["Ansible 基本架构"](https://img2018.cnblogs.com/blog/1356806/201909/1356806-20190908123220930-621197221.png "Ansible 基本架构")


- 核心引擎：即 Ansible。
- 核心模块（core modules）：Ansible 模块资源分发到远程节点使其执行特定任务或匹配一个特定的状态。
- 自定义模块（custom modules）
- 插件（plugins）：模块功能的补充，借助插件完成记录日志、邮件等功能。
- 剧本（playbook）：定义 Ansible 任务的配置文件，可将多个任务定义在一个剧本中，由 Ansible 自动执行，可由控制主机运行多个任务，同时对多台远程主机进行管理。
- 连接插件（connectior plugins）：Ansible 基于连接插件连接到各个主机，负责和被管节点实现通信。因为支持除SSH连接方法外的其他连接方法，所以需要连接插件。
- 主机清单（host inventory）：定义 Ansible 管理的主机策略。

Ansible 采用 paramiko 协议库，通过 SSH 或 ZeroMQ 等连接主机。Ansible 在控制主机将 Ansible 模块通过 SSH协议推送到被管节点执行，执行完自动删除。控制主机与被管节点之间支持 local、SSH、ZeroMQ 三种连接方式，默认使用基于 SSH 连接，在大规模情况下，使用 ZeroMQ 连接方式执行速度更快。

### 任务执行模式
Ansible 系统由控制主机对被管节点的操作方式可分为两类，即 ad-hoc 和 playbook。
- ad-hoc 模式使用单个模块，支持批量执行单条命令。
- playbook 模式是 Ansible 主要管理方式，playbook 通过多个 task 集合完成一类功能。（可以把 playbook 理解为通过组合多条ad-hoc 操作的配置文件）

!["Ansible 执行过程"](https://img2018.cnblogs.com/blog/1356806/201909/1356806-20190908124818630-813126839.png "Ansible 执行过程")

## Ansible 特性
Ansible 是基于一致性、安全性、高可靠性设计的轻量级自动化工具，具有功能强大、部署便捷、描述清晰等特性，很好地解决了统一配置、统一部署、流程编排等复杂的 IT 自动化管理问题。
### Ansible 功能特性
1. 语法简单、易读
2. 不需要再被管节点安装客户端软件
3. 基于推送（Push）方式
4. 方便管理小规模场景
5. 大量内置模块
6. 非常轻量级的抽象层

### Ansible 与其他配置管理的对比
项目 | Puppet | SaltStack | Ansible
---|---|---|---
开发语言 | Ruby | Python | Python
是否有客户端 | 有 | 有 | 无
是否支持二次开发 | 不支持 | 支持 | 支持
服务器与远程机器通信协议 | 标准 SSL 协议 | 使用AES加密 | 使用 OpenSSH
配置文件格式 | Ruby语法格式 | YAML | YAML

与其他自动化工具比较，Ansible 不需要安装客户端就可以轻松地管理、配置。
!["Ansible 自动化运维方式"](https://img2018.cnblogs.com/blog/1356806/201909/1356806-20190908124934363-928463580.png "Ansible 自动化运维方式")

## 总结
Ansible 的关键想法是计算机是一组，而不是一个个分开的机器，即“多层编排”的思想。避免了证书交换，以及反向解析 DNS 和 NTP 的问题。YAML的配置文件格式，简单易用。