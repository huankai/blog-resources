---
title: Centos 内核参数优化
date: {{ date }}
author: huangkai
tags:
    - Linux
---


# 一、配置 ulimit 参数 #

查看参数值
```
[root@iZwz95lasa7ruhzh7ftd5bZ ~]# ulimit -n
1024
```

配置:
vim /etc/security/limits.conf

```
* soft nofile 102400
* hard nofile 102400
* soft nofile 102400
* hard nofile 102400
```