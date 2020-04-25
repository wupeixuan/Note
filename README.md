- [文章目录](#文章目录)
    * [Java](#java)
    * [ElasticSearch](#elasticsearch)    
    * [Nginx](#nginx)
    * [MySQL](#mysql)
    * [Redis](#redis)
    * [ZooKeeper](#zookeeper)
    * [微服务](#微服务)
    * [网络](#网络)
    * [设计模式](#设计模式)
    * [Spring Boot](#spring-boot)
    * [运维](#运维)
    * [Git](#git)
    * [消息队列](#消息队列)
    * [源码解析](#源码解析)
    * [面经](#面经)
    * [算法](#算法)
    * [程序人生](#程序人生)
    * [工具](#工具)

欢迎进入我的个人博客：[武培轩的博客](https://www.tianheyu.top)

**微信公众号**

![微信公众号](https://img-blog.csdnimg.cn/20200110151256933.png)

专注分享后端技术干货，包括 Java 基础、Java 并发、JVM、Nginx、Zookeeper、ElasticSearch、微服务、消息队列、源码解析、数据库、设计模式、面经等，助你编程之路少走弯路。

**后端技术交流群**

有一句话说得很好，一个人学习可以走得很快，但一群人学习可以走得更远。

所以，如果你想和众多优秀的人一起学习，可以考虑加入技术交流群。扫描微信二维码，备注【加群】添加好友，我会迅速拉你进群。

![微信二维码](https://img-blog.csdnimg.cn/202001101531450.jpg)

# 文章目录

## Java

[给学妹的 Java 学习路线](https://mp.weixin.qq.com/s/5FY87UBkqXpYvk2BiXlJ3Q)

[一男子给对象转账5000元，居然又退还了！](https://mp.weixin.qq.com/s/ppIJnBj66pnBzHE_Za81TA)

[我就站在你面前，你却视而不见！](https://mp.weixin.qq.com/s/AIcwp6wWIs7MPyxXA7NagA)

[编译器：人家就要乱来！](https://mp.weixin.qq.com/s/f0p3vqYRacSDF728oXJxQA)

[2020 年 Java 程序员应该学习什么？](https://mp.weixin.qq.com/s/Oz_mm6RbSvQPjyDAvpQt4w)

[为什么阿里巴巴Java开发手册中不建议在循环体中使用+进行字符串拼接？](https://mp.weixin.qq.com/s/dc7HW0SqEcJknMIqoQZnlg)

[为什么阿里巴巴Java开发手册中强制要求整型包装类对象值用 equals 方法比较？](https://mp.weixin.qq.com/s/uP9Jy75OlwavPGwVabOl8A)

[为什么阿里巴巴Java开发手册中强制要求不要在foreach循环里进行元素的remove和add操作？](https://mp.weixin.qq.com/s/q4S6dCIwtDbuQ4KxSBItqA)

[最大的 String 字符长度是多少？](https://mp.weixin.qq.com/s/MLoN0SZc0s_H3GcGMfoYhA)

[一文搞懂 ThreadLocal 原理](https://mp.weixin.qq.com/s/HU5PTysA6gndCpYcrb-KCg)

[为什么不建议使用Date，而是使用Java8新的时间和日期API？](https://mp.weixin.qq.com/s/eMtrhOMuXpvs3s_o9yA99Q)

[在 Java 中如何比较日期？](https://mp.weixin.qq.com/s/o2hB2EOfdXbcu8aLbRip4Q)

[Java 8 Optional：优雅地避免 NPE](https://mp.weixin.qq.com/s/NaGekMgUMwXauQ-Zkx46Bg)

[深入理解 Java 中的 final 关键字](https://mp.weixin.qq.com/s/bTEziv7CAbTXcdeKiJaeRg)

[Java 中的 final、finally、finalize 有什么不同？](https://mp.weixin.qq.com/s/2i4LDFrVjESCsB1XsyW-Hw)

[Java中Set集合是如何实现添加元素保证不重复的？​](https://mp.weixin.qq.com/s/87PVnhkWDQuW57ahnD28bg)

[你真的了解 volatile 关键字吗？](https://mp.weixin.qq.com/s/35iBa26Y8XLlCsYQzVoHsg)

[你编写的Java代码是咋跑起来的？](https://mp.weixin.qq.com/s/PyMu-fH6fpSdfjCU4Sq_Jw)

[Java线程的生老病死](https://mp.weixin.qq.com/s/4EDzHrAh0UUtfr4xBNXBiA)

[如何优雅地中止线程？](https://mp.weixin.qq.com/s/glkgYlArGqo16S8UKXS9Ag)

[线程数，射多少更舒适？](https://mp.weixin.qq.com/s/zgFg6oPabcqoZ-X2_nX9eQ)

[如何优雅地中止线程？](https://mp.weixin.qq.com/s/glkgYlArGqo16S8UKXS9Ag)

[原来 CPU 为程序性能优化做了这么多](https://mp.weixin.qq.com/s/ZzdNlbhgjGk6iTXHVIZxsg)

[Java异常处理只有Try-Catch吗？](https://mp.weixin.qq.com/s/zFkE-7jR6VK-b1C2rPJWWA)

[如何编写可怕的Java代码？](https://mp.weixin.qq.com/s/MXSaSvkvTbOkDUCcQEmQaw)

[请停止编写这么多的for循环！](https://mp.weixin.qq.com/s/7vn0ZSOkInK7oyPwh3SaQA)

[JVM内存模型](https://mp.weixin.qq.com/s/aKDKX6qge7-pLxX754kHdw)

[JVM GC算法](https://mp.weixin.qq.com/s/P40Vu0eY-xOkNRSn7Ue9ow)

[JVM类加载过程与双亲委派模型](https://mp.weixin.qq.com/s/mUCmntvFK5CoWOTcXUUI5A)

[Full GC 和 Minor GC，傻傻分不清楚](https://mp.weixin.qq.com/s/zlc2UKMoYlFrUcGVb3_8rw)

[请停止编写这么多的for循环！](https://mp.weixin.qq.com/s/7vn0ZSOkInK7oyPwh3SaQA)

## ElasticSearch

[全文搜索引擎 Elasticsearch 入门：集群搭建](https://mp.weixin.qq.com/s/NrN2Vcj8Evpt8LCMPrUgPw)

[手把手教你搭建 ELK 实时日志分析平台](https://mp.weixin.qq.com/s/jIXh0DIYZl_9cGP1RBNbuQ)

[一篇文章带你搞定 ElasticSearch 术语](https://mp.weixin.qq.com/s/tsoBovXDcB02KxvWu2_SpQ)

[搜索引擎之倒排索引浅析](https://mp.weixin.qq.com/s/Kp-KDUSFosEJOIhekw4wHw)

[ElasticSearch 分词器，了解一下](https://mp.weixin.qq.com/s/5mXFIvcJvP3RiadkpQtrpA)

[ElasticSearch 文档的增删改查都不会？](https://mp.weixin.qq.com/s/q-LUgsZS-fz8DHvznTIiJA)

[看完这篇还不会 Elasticsearch 搜索,那我就哭了！](https://mp.weixin.qq.com/s/SHHwh-1iPhfOv7qrqWVRJw)

[一文搞懂 Elasticsearch 之 Mapping](https://mp.weixin.qq.com/s/y7hm4-vzGd2X_ok3sp_6GQ)

## Nginx

[Nginx 了解一下？](https://mp.weixin.qq.com/s/FX1w12GFtceqpd48BOC6bA)

[Nginx 热部署和日志切割，你学会了吗？](https://mp.weixin.qq.com/s/KNEEMy8DGbpOAf-wXxumgA)

[使用 Nginx 搭建静态资源 web 服务器](https://mp.weixin.qq.com/s/8nX5fwoyjG06Sy7kLf9-sg)

[Nginx 的请求处理流程，你了解吗？](https://mp.weixin.qq.com/s/AvYXDMy3TqkXyDXvWXeOWw)

[Nginx 的进程结构，你明白吗？](https://mp.weixin.qq.com/s/ZwpQPcJECDOTdVpidLvLbw)

[Nginx 进程管理，你需要了解哪些？](https://mp.weixin.qq.com/s/CgWlHvBoOrXXqkzifrgKkQ)

[探究 Nginx 中 reload 流程的真相](https://mp.weixin.qq.com/s/zscL0h-VTVUPAX55BsyFcA)

[Nginx热升级流程，看这篇就够了](https://mp.weixin.qq.com/s/_fTRj9bSASIrv4O4Swtv_w)

[如何优雅地关闭worker进程？](https://mp.weixin.qq.com/s/Zk9LlXsyqKmB_A9GPQ5rGQ)

[浅析 Nginx 网络事件](https://mp.weixin.qq.com/s/sNrZPbGxN-YLCOz0RocqxA)

[Nginx 究竟如何处理事件？](https://mp.weixin.qq.com/s/2o_lzhyIvuYm1l9wJSCWPA)

[一文搞懂 Elasticsearch 之 Mapping](https://mp.weixin.qq.com/s/y7hm4-vzGd2X_ok3sp_6GQ)

[Elasticsearch 之聚合分析入门](https://mp.weixin.qq.com/s/0h4qzSvhkKrLgLQh59dk7Q)

## MySQL

[数据库事务的四大特性以及隔离级别](https://mp.weixin.qq.com/s/Y-qYIEShHCEKYQHZnMLSeQ)

[一条SQL查询语句是如何执行的？](https://mp.weixin.qq.com/s/jWw07LxQooHdv_4DvPLoyQ)

[MySQL 日志系统之 redo log 和 binlog](https://mp.weixin.qq.com/s/g-QHcctt_fOmJmQQI3ISOQ)

## Redis

[Redis 系列（一）五种基本数据结构](https://mp.weixin.qq.com/s/OzGqfjot7iC6UARvAkC1wQ)

[Redis 系列（二）跳跃表](https://mp.weixin.qq.com/s/CCJ4c-1y4nqH2-GTQ1J1oQ)

[Redis 系列（三）分布式锁深入探究](https://mp.weixin.qq.com/s/qIXOYNx776_rGwoG8vblHg)

[Redis 系列（四）统计问题看这篇就够了](https://mp.weixin.qq.com/s/6S8zi8jdQDqgcgUF6XWRDw)

[Redis持久化](https://mp.weixin.qq.com/s/VTJHY2-VPy6OeaQmkwdiAw)

## ZooKeeper

[ZooKeeper 入门看这篇就够了](https://mp.weixin.qq.com/s/GZHOR4s-n0tjTSoYpgTD6A)

[一篇文章带你了解 ZooKeeper 架构](https://mp.weixin.qq.com/s/8MzJbqoeNNYrwbopRKDi9Q)

## 微服务

[什么是微服务？](https://mp.weixin.qq.com/s/tKhdhEsJ5dHZIfxow9Wgew)

[从单体应用走向服务化](https://mp.weixin.qq.com/s/0wgbld5T9WtHISmVWDFoYA)

[初探微服务架构](https://mp.weixin.qq.com/s/prtD_7cZFMgV874p9W6SRQ)

## 网络

[TCP三次握手和四次挥手](https://mp.weixin.qq.com/s/aZkbHrImIOpY7VG6vcD9cg)

[当你在浏览器地址栏输入一个URL后回车，将会发生的事情？](https://mp.weixin.qq.com/s/Cm049VWAPkkaGW6iz40wTg)

[Session 和 Cookie 区别](https://mp.weixin.qq.com/s/o_JEYz33YRnT8crJgTt3vQ)

[TCP和UDP的区别](https://mp.weixin.qq.com/s/0G6F6q_BpouRjZ2mf6XGww)

[HTTP0.9 HTTP1.0 HTTP 1.1 HTTP 2.0区别](https://mp.weixin.qq.com/s/S929mozh1b3Mz1rS_VKDVg)

## 设计模式

[设计模式-单例模式](https://mp.weixin.qq.com/s/jjZwBMSeIYk_sx8Hy_RrJg)

[设计模式-代理模式](https://mp.weixin.qq.com/s/hTKPg14oJLj9y4RlDc6n7Q)

[设计模式-观察者模式](https://mp.weixin.qq.com/s/R9bnY9WtZEMGHZxP6ImWaQ)

[设计模式-简单工厂模式](https://mp.weixin.qq.com/s/wFcUa9U5R1cc0syR8LVF8w)

[设计模式-工厂方法模式](https://mp.weixin.qq.com/s/qzwfAA8t5BwP6xf5IyDthQ)

[设计模式-抽象工厂模式](https://mp.weixin.qq.com/s/PpiwCsgw2mt3OWd0nsjxhg)

## Spring Boot

[如何定制 Spring Boot 的 Banner？](https://mp.weixin.qq.com/s/ltMTsrl6fGNLSCq9UDYQxA)

[Spring Boot 定时任务 @Scheduled](https://mp.weixin.qq.com/s/ILMQioj2ihzSeArli3M04w)

[深入源码分析SpringMVC执行过程](https://mp.weixin.qq.com/s/jaWDPpiCuSPdhLVLVSyYcw)

## 运维

[每个开发人员都应该知道的11个Linux命令](https://mp.weixin.qq.com/s/ZPz_rvXPgn3NBGXWVViouQ)

[Ansible自动化运维-Ansible架构及特点](https://mp.weixin.qq.com/s/DxsEqXyzDRK9fHDAmlT9iw)

[Ansible自动化运维-Ansible安装与配置](https://mp.weixin.qq.com/s/9w2gsRLD0Bl9eKNXT9_Kbw)

[Ansible自动化运维-Ansible组件介绍](https://mp.weixin.qq.com/s/qlToA4_1k5XepUHX5fXljA)

## Git

[请停止编写糟糕的提交消息！](https://mp.weixin.qq.com/s/VItk9iKdXitjSYkVCTxq2g)

[看完这篇还不会用Git，那我就哭了！](https://mp.weixin.qq.com/s/JOCfkKPQE3UvwhP7h9_UmA)

## 消息队列

[如何选择消息队列？](https://mp.weixin.qq.com/s/08dgGSbEK2rAzPVrjklyrQ)

## 源码解析

[Apollo源码解析-搭建调试环境](https://mp.weixin.qq.com/s/jdJK8gRFMQ-cMlil8lvoCw)

[HashMap源码解析](https://mp.weixin.qq.com/s/Ixzpclg70AiR21LX4rlZng)

## 面经

[京东面经汇总](https://mp.weixin.qq.com/s/HCCX3V6R2f7q3cdGa3qXFQ)

[小米面经汇总](https://mp.weixin.qq.com/s/HIqqUdHr7yS17AsBTW9q1w)

[答完这10道题，我哭了](https://mp.weixin.qq.com/s/4XBelNjgwAynsuJYUmjlMw)

## 算法

[什么是数据结构？](https://mp.weixin.qq.com/s/STLpfs6dOVmeWaOW6mIO8A)

[什么是链表？](https://mp.weixin.qq.com/s/ZuCcYvUtrcvJr19tDzrNnw)

[什么是数组？](https://mp.weixin.qq.com/s/AeiLlcBMRGf2HYxy7AcBrg)

[什么是栈？](https://mp.weixin.qq.com/s/NPGlJ1TjHX6ZH0KNsULYbA)

[什么是队列？](https://mp.weixin.qq.com/s/mvROTx6iHmJofXZj9Qvl4A)

[什么是哈希表？](https://mp.weixin.qq.com/s/aJPcKst7bgDyQo2oBXe_cw)

[剑指Offer-重建二叉树](https://mp.weixin.qq.com/s/3VqOJ0SrivlgAMPWiKEJqg)

[剑指Offer-把二叉树打印成多行](https://mp.weixin.qq.com/s/rJQrF6-0B6LdSoNgd4E72g)

[剑指Offer-把数组排成最小的数](https://mp.weixin.qq.com/s/sYtc_04LWLSHHcQJWwujBg)

[剑指Offer-求1+2+3+...+n](https://mp.weixin.qq.com/s/WYIlga6jcHTeC9nMzq1XiA)

[剑指Offer-用两个栈实现队列](https://mp.weixin.qq.com/s/ajFZiKVc6E6NXKwDfRKcSA)

## 程序人生

[程序员的圈子，就差你了！！！](https://mp.weixin.qq.com/s/eaCY8z9EwnT9S1q4Xf3fFg)

[后端技术交流社群](https://mp.weixin.qq.com/s/FNJ406BKfa1zh7VU42uXIg)

[给初学者的技巧，只有3条，不看后悔](https://mp.weixin.qq.com/s/IQ6l2ntkInHeQRY_DSE1YA)

[9 个习惯助你在新的一年更有精力](https://mp.weixin.qq.com/s/oz2cED1CexLrPPctgHwZWA)

[如何优雅地在Stack Overflow提问？](https://mp.weixin.qq.com/s/INPm6eS4OhtAH8n5vI8psg)

[代码重构有什么意义？为什么重构有用？](https://mp.weixin.qq.com/s/1IyWSehaHbUb5Cj0llOeJw)

## 工具

[IDEA到期了？不用怕，最新的永久激活送给你](https://mp.weixin.qq.com/s/3Ksc7rnVo4n6bu2k2M1bwQ)

[听说用 Lombok 可以早点下班？](https://mp.weixin.qq.com/s/R720b6dlYntrKSsswos6eg)

[后缀补全用得好，提前下班没烦恼](https://mp.weixin.qq.com/s/qDtc46iNq_s0VpQbLyBZXw)