---
title: clustershell 使用
date: {{ date }}
author: huangkai
tags:
    - Linux
---


介绍
==

在运维实战中，如果有若干台数据库服务器，想对这些服务器进行同等动作，比如查看它们当前的即时负载情况，查看它们的主机名，分发文件等等，这个时候该怎么办？一个个登陆服务器去操作，太傻帽了！写个shell去执行，浪费时间~~

这种情况下，如果集群数量不多的话，选择一个轻量级的集群管理软件就显得非常有必要了。ClusterShell就是这样一种小的集群管理工具，原理是利用ssh，可以说是Linux系统下非常好用的运维利器！

安装：
---

```
[huangkai@100 ~]$ sudo yum -y install clustershell
```