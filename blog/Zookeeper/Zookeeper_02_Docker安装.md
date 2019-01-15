---
title: Zookeeper_02 Docker集群安装
date: {{ date }}
author: huangkai
tags:
    - Zookeeper
---

# 一、安装环境： #

|服务器|ip|
|:-:|:-:| 
|sjq128|192.168.64.128|
|sjq129|192.168.64.129|
|sjq130|192.168.64.130|

# 二、docker 安装： #

- 安装docker:
点击 [这里](https://github.com/huankai/blog-resources/blob/master/blog/Docker/Docker_01_%E7%AE%80%E4%BB%8B%E4%B8%8E%E5%AE%89%E8%A3%85.md) 查看。
- 安装docker-compose:
使用root 账号执行：
```
[root@localhost ~]# curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
	[root@localhost ~]# chmod +x /usr/local/bin/docker-compose
```

# 三、配置： #

- sjq128

```
version: "3"
services:
  zookeeper:
    image: zookeeper
	restart: "always"
	hostname: sjq128
	container_name: zookeeper
	volumes:
	  - "/etc/timezone:/etc/timezone:ro"
	  - "/etc/localtime:/etc/localtime:ro"
	ports:
	  - "2181:2181"
	  - "2888:2888"
	  - "3888:3888"
	environment:
	  ZOO_MY_ID: 1
	  ZOOKEEPER_PORT: 2181
	  ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=sjq129:2888:3888 server.3=sjq130:2888:3888
	extra_hosts:
	  - "sjq128:192.168.64.128"
	  - "sjq129:192.168.64.129"
	  - "sjq130:192.168.64.130"
```
- sjq129

```
version: "3"
services:
  zookeeper:
    image: zookeeper
	restart: "always"
	hostname: sjq129
	container_name: zookeeper
	volumes:
	  - "/etc/timezone:/etc/timezone:ro"
	  - "/etc/localtime:/etc/localtime:ro"
	ports:
	  - "2181:2181"
	  - "2888:2888"
	  - "3888:3888"
	environment:
	  ZOO_MY_ID: 2	
	  ZOOKEEPER_PORT: 2181
	  ZOO_SERVERS: server.1=server.1=sjq128:2888:3888 server.2=0.0.0.0:2888:3888 server.3=sjq130:2888:3888
	extra_hosts:
	  - "sjq128:192.168.64.128"
	  - "sjq129:192.168.64.129"
	  - "sjq130:192.168.64.130"
```
- sjq130

```
version: "3"
services:
  zookeeper:
    image: zookeeper
	restart: "always"
	hostname: sjq130
	container_name: zookeeper
	volumes:
	  - "/etc/timezone:/etc/timezone:ro"
	  - "/etc/localtime:/etc/localtime:ro"
	ports:
	  - "2181:2181"
	  - "2888:2888"
	  - "3888:3888"
	environment:
	  ZOO_MY_ID: 3
	  ZOOKEEPER_PORT: 2181
	  ZOO_SERVERS: server.1=sjq128:2888:3888 server.2=sjq129:2888:3888 server.3=0.0.0.0:2888:3888
	extra_hosts:
	  - "sjq128:192.168.64.128"
	  - "sjq129:192.168.64.129"
	  - "sjq130:192.168.64.130"
```


如上配置中，每台服务器连接自己的zookeeper 容器要配置为 `0.0.0.0` ，不能配置为宿主机的ip地址，否则不能启动成功。
参考资料 https://stackoverflow.com/questions/30940981/zookeeper-error-cannot-open-channel-to-x-at-election-address
# 四、启动： #
防火墙开放 2181/2888/3888 端口。

分别在三台服务器执行 `docker-compose up -d zookeeper` 启动 zookeeper

# 五、进入zookeeper 容器，查看启动状态 #

```
[huangkai@sjq130 docker]$ docker-compose exec zookeeper bash
bash-4.4# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Mode: leader
bash-4.4# 
```