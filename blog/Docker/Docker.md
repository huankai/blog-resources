---
title: Docker
date: {{ date }}
author: huangkai
tags:
    - Docker
---

# 一、介绍 #

# 二、安装 #
##  2.1、安装环境： ##
操作系统：CentOS Linux release 7.3.1611 (Core)
Docker版本：Docker version 1.12.6, build 3e8e77d/1.12.6
- yum 安装
```
[root@sjq01 ~]$ yum -y install docker
```

## 2.2、启动 ： ##
```
[root@sjq01 ~]$ systemctl start docker.service
```

## 2.4、停止 ： ##
```
[root@sjq01 ~]$ systemctl stop docker.service
```

## 2.4、配置国内镜像： ##
Docker默认的镜像为 https://hub.docker.com/ ，从此镜像下载会非常慢，可根据如下配置使用国内镜像下载
```
[root@sjq01 ~]$ vim /etc/docker/daemon.json
# 添加内容如下：

{
"registry-mirrors": ["https://registry.docker-cn.com"]
}

```
重启Docker:
配置完之后执行下面的命令，以使docker的配置文件生效
```
[root@sjq01 ~]$systemctl daemon-reload 
[root@sjq01 ~]$systemctl restart docker
```