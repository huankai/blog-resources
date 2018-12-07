---
title: Zookeeper_01介绍与安装
date: {{ date }}
author: huangkai
tags:
    - Zookeeper
---

# 一、Zookeeper 简介 #


# 二、Zookeeper 安装 #
部署环境:

192.168.64.128、192.168.64.129、 192.168.64.130 三台服务器

## 1、修改 host ##

每台服务器修改 /etc/hosts，添加内容如下，修改完成后，使用 ping 命令检查是否能通(如：ping sjq-01)。
```
[huangkai@sjq-02 conf]$ sudo vim /etc/hosts

192.168.64.128  sjq-01
192.168.64.129  sjq-02
192.168.64.130  sjq-03
```

## 2、下载、解压与安装 zookeeper ##
下载地址：https://archive.apache.org/dist/zookeeper/

解压、安装：

```
[huangkai@sjq-02 ~]$sudo tar -zxvf zookeeper-3.4.9.tar.gz -C /usr/local/
[huangkai@sjq-02 ~]$ sudo chown huangkai:huangkai /usr/local/zookeeper-3.4.9 -R
[huangkai@sjq-02 ~]$ cd /usr/local/zookeeper-3.4.9
[huangkai@sjq-02 zookeeper-3.4.9]$ mkdir data dataLog
[huangkai@sjq-02 zookeeper-3.4.9]$ cd data
[huangkai@sjq-02 zookeeper-3.4.9]$ vim myid #每台服务器在此文件中输入唯一的编号，如128服务器输入 1 ，129服务器输入2,130服务器输入3 ，保存退出
[huangkai@sjq-02 zookeeper-3.4.9]$ cd conf
[huangkai@sjq-02 zookeeper-3.4.9]$ mv zoo_sample.cfg zoo.cfg #修改文件名
[huangkai@sjq-02 zookeeper-3.4.9]$ vim zoo.cfg  # 三台服务器输入内容相同，如下：

tickTime=2000

initLimit=10

syncLimit=5

dataDir=/usr/local/zookeeper-3.4.9/data

dataLogDir=/usr/local/zookeeper-3.4.9/dataLog

clientPort=2181

server.1=sjq-01:2888:3888 
server.2=sjq-02:2888:3888 
server.3=sjq-03:2888:3888

```

启动 zookeeper:
```
[huangkai@sjq-01 zookeeper-3.4.9]$ ./bin/zkServer.sh start
```
查看zookeeper 状态 ：

192.168.64.128(如下，角色为 follower):

```
[huangkai@sjq-01 zookeeper-3.4.9]$ ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: follower
[huangkai@sjq-01 zookeeper-3.4.9]$ 
```

192.168.64.129(如下，角色为 leader):

```
[huangkai@sjq-02 zookeeper-3.4.9]$ ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: leader
[huangkai@sjq-02 zookeeper-3.4.9]$ 
```

192.168.64.130(如下，角色为 follower):

```
[huangkai@sjq-03 zookeeper-3.4.9]$ ./bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: follower
[huangkai@sjq-03 zookeeper-3.4.9]$
```

如上，zookeeper集群启动成功。

# 三、Zookeeper 配置开机启动 #

```
将zookeeper目录所有者给指定用户 huangkai
[huangkai@sjq-01 ~]$ chown huangkai:huangkai /usr/local/zookeeper-3.4.9/ -R
配置zookeeper 目录当前用户可执行权限
[huangkai@sjq-01 ~]$ chmod u+x /usr/local/zookeeper-3.4.9/ -R
```

配置zookeeper日志目录 ：


1、修改 zoo.cfg 配置，添加内容如下：

```
dataLogDir=/usr/local/zookeeper-3.4.9/logs
```
2、创建日志目录 ：

```
[huangkai@sjq-01 zookeeper-3.4.9]$ mkdir logs
```

3、[huangkai@sjq-01 zookeeper-3.4.9]$ vim ./bin/zkServer.sh

```
....
# 大约在 125行左右：添加如下一句话：
ZOO_LOG_DIR="$($GREP "^[[:space:]]*dataLogDir" "$ZOOCFG" | sed -e 's/.*=//')"


if [ ! -w "$ZOO_LOG_DIR" ] ; then
mkdir -p "$ZOO_LOG_DIR"
fi

_ZOO_DAEMON_OUT="$ZOO_LOG_DIR/zookeeper.out"
....

```

编写 zookeeper.service文件如下：

```
[Unit]
Description=zookeeper
After=zookeeper.service

[Service]
Type=forking
User=huangkai
Group=huangkai
PIDFile=/usr/local/zookeeper-3.4.9/data/zookeeper_server.pid
ExecStart=/usr/local/zookeeper-3.4.9/bin/zkServer.sh start
ExecReload=/usr/local/zookeeper-3.4.9/bin/zkServer.sh restart
ExecStop=/usr/local/zookeeper-3.4.9/bin/zkServer.sh stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
# 三、zookeeper 管理工具 #

3.1、 [zkUI](https://github.com/DeemOpen/zkui)
3.2、[zookeeper-dev-ZooInspector](https://issues.apache.org/jira/secure/attachment/12436620/ZooInspector.zip)
