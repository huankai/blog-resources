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
|node100|192.168.117.100|
|node101|192.168.117.101|
|node102|192.168.117.102|

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
<font color='red'>如下中的environment ZOO_SERVERS 如果某个节点配置的是 observer 角色，可以使用 `ip:port:observer;2181` 配置，此时，此节点只是 observer 角色，不参与 leader 投票选举，参考 : https://zookeeper.apache.org/doc/r3.5.5/zookeeperReconfig.html </font>

- node100

```
version: "3.8"
services:
  zookeeper:
    image: zookeeper:3.5
    container_name: zookeeper
    hostname: node100
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - "/www/docker/zookeeper/data:/data"
      - "/www/docker/zookeeper/datalog:/datalog"
    ports:
      - "2181:2181"
      - "2888:2888"
      - "3888:3888"
    environment:
      ZOO_MY_ID: 1
	  ZOO_STANDALONE_ENABLED: "false" # 默认为true
      ZOO_ADMINSERVER_ENABLED: "false" #是否开启 admin 模式，默认为true,admin 模式使用 http 8080 端口，可以使用 http://${ip}:8080/commands/stat
      ZOO_AUTOPURGE_PURGEINTERVAL: 48  
# 这个参数和 ZOO_AUTOPURGE_PURGEINTERVAL 搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个
      ZOO_AUTOPURGE_SNAPRETAINCOUNT: 60 
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=node101:2888:3888;2181 server.3=node102:2888:3888;2181
    extra_hosts:
      - "node100:192.168.117.100"
      - "node101:192.168.117.101"
      - "node102:192.168.117.102"
    restart: always
```
- node101

```
version: "3.8"
services:
  zookeeper:
    image: zookeeper:3.5
    container_name: zookeeper
    hostname: node101
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - "/www/docker/zookeeper/data:/data"
      - "/www/docker/zookeeper/datalog:/datalog"
    ports:
      - "2181:2181"
      - "2888:2888"
      - "3888:3888"
    environment:
      ZOO_MY_ID: 2
      ZOO_STANDALONE_ENABLED: "false"
      ZOO_ADMINSERVER_ENABLED: "false"
      ZOO_AUTOPURGE_PURGEINTERVAL: 48  
# 这个参数和 ZOO_AUTOPURGE_PURGEINTERVAL 搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个
      ZOO_AUTOPURGE_SNAPRETAINCOUNT: 60 
      ZOO_SERVERS: server.1=node100:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=node102:2888:3888;2181
    extra_hosts:
      - "node100:192.168.117.100"
      - "node101:192.168.117.101"
      - "node102:192.168.117.102"
    restart: always
```
- node102

```
version: "3.8"
services:
  zookeeper:
    image: zookeeper:3.5
    container_name: zookeeper
    hostname: node102
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - "/www/docker/zookeeper/data:/data"
      - "/www/docker/zookeeper/datalog:/datalog"
    ports:
      - "2181:2181"
      - "2888:2888"
      - "3888:3888"
    environment:
      ZOO_MY_ID: 3
      ZOO_STANDALONE_ENABLED: "false"
      ZOO_ADMINSERVER_ENABLED: "false"
# 单位是小时，需要填写一个1或更大的整数，默认是0，表示不开启自己清理功能
      ZOO_AUTOPURGE_PURGEINTERVAL: 48  
# 这个参数和 ZOO_AUTOPURGE_PURGEINTERVAL 搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个
      ZOO_AUTOPURGE_SNAPRETAINCOUNT: 60 
      ZOO_SERVERS: server.1=node100:2888:3888;2181 server.2=node101:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
    extra_hosts:
      - "node100:192.168.117.100"
      - "node101:192.168.117.101"
      - "node102:192.168.117.102"
    restart: always
```


如上配置中，每台服务器连接自己的zookeeper 容器要配置为 `0.0.0.0` ，不能配置为宿主机的ip地址，否则不能启动成功。
参考资料 https://stackoverflow.com/questions/30940981/zookeeper-error-cannot-open-channel-to-x-at-election-address
# 四、启动： #
防火墙开放 2181/2888/3888 端口。

分别在三台服务器执行 `docker-compose up -d zookeeper` 启动 zookeeper

# 五、进入zookeeper 容器，查看启动状态 #

```
[huangkai@l101 docker]$ docker-compose exec zookeeper bash
root@node101:/apache-zookeeper-3.5.5-bin# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader
root@node101:/apache-zookeeper-3.5.5-bin# 
```

**问题：**
1、在某个从节点上使用 `zkServer.sh status` 查看状态时，出现如下：
```
root@node100:/apache-zookeeper-3.5.5-bin# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /conf/zoo.cfg
Client port found: 2181. Client address localhost.
Error contacting service: It is probably not running.
```
并且使用docker 查看日志信息时，答应如下日志信息：
```
[root@node100 docker]# docker-compose logs -f --tail 100 zookeeper
zookeeper			   | 2019-07-23 10:12:32,800 [myid:1] - INFO  [QuorumPeer[myid=1](plain=/0.0.0.0:2181)(secure=disabled):QuorumCnxManager@430] - Have smaller server identifier, so dropping the connection: (3, 1)
zookeeper               | 2019-07-23 10:12:32,800 [myid:1] - INFO  [QuorumPeer[myid=1](plain=/0.0.0.0:2181)(secure=disabled):FastLeaderElection@919] - Notification time out: 51200
zookeeper               | 2019-07-23 10:13:24,002 [myid:1] - INFO  [QuorumPeer[myid=1](plain=/0.0.0.0:2181)(secure=disabled):QuorumCnxManager@430] - Have smaller server identifier, so dropping the connection: (2, 1)
zookeeper               | 2019-07-23 10:13:24,004 [myid:1] - INFO  [QuorumPeer[myid=1](plain=/0.0.0.0:2181)(secure=disabled):QuorumCnxManager@430] - Have smaller server identifier, so dropping the connection: (3, 1)
zookeeper               | 2019-07-23 10:13:24,004 [myid:1] - INFO  [QuorumPeer[myid=1](plain=/0.0.0.0:2181)(secure=disabled):FastLeaderElection@919] - Notification time out: 60000
```
解决方法：重新启动 leader 节点
参考资料：https://issues.apache.org/jira/browse/ZOOKEEPER-2938

2、按上面的方式启动后，使用 `zkCli.sh` 连接，执行 `ls /` 报错Session 失效，zk ui 工具也不能连接zookeeper，重新启动所有节点即可


六、zkui docker 安装
================

6.1、下载源码：
	https://github.com/DeemOpen/zkui

解压，使用maven 打包：
```	
mvn clean package -Dtest.skip=true
```
6.2、创建 zkui 目录
```
[huangkai@100 docker ]$ mkdir -p /www/docker/zookeeper-ui/config

```
6.3、将下载下的源码中的 `bootstrap.sh` 文件上传到 `/www/docker/zookeeper-ui` 目录下

6.4、编写 Dockerfile
vim /www/docker/zookeeper-ui/Dockerfile ,部分摘自 源码中的 Dockerfile  
```

FROM java:8

MAINTAINER Miguel Garcia Puyol <miguelpuyol@gmail.com>

WORKDIR /var/app

ADD zkui-*.jar /var/app/zkui.jar
ADD bootstrap.sh /var/app/bootstrap.sh
RUN chmod u+x /var/app/bootstrap.sh
ENTRYPOINT ["/var/app/bootstrap.sh"]

EXPOSE 9090
```
6.5、复制源码根目录下的 `config.cfg `文件到 `/www/docker/zookeeper-ui/config` 中，修改 zookeeper 地址

```
....
zkServer=192.168.117.100:2181,192.168.117.101:2181,192.168.117.102:2181
...
```

6.6、修改 docker-compose.yml 配置 zookeeper-ui 
vim /www/docker/zookeeper-ui/docker-compose.yml

```
version: "3.8"
services:
  zookeeper-ui:
    build: ./zookeeper-ui
    container_name: zookeeper-ui
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - "/www/docker/zookeeper-ui/config/config.cfg:/var/app/config.cfg"
    ports:
      - "9090:9090"
    restart: always
```

6.7、启动 zookeeper-ui

```
docker-compose up -d zookeeper-ui
```