---
title: Kafka_01介绍与安装
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

# 一、下载与安装： #

## 1.1、 下载： ##

官网下载地址： http://kafka.apache.org/downloads

使用清华大学镜像地址下载：

[huangkai@sjq-10 install]$ **wget http://mirror.bit.edu.cn/apache/kafka/2.0.0/kafka_2.12-2.0.0.tgz**

也可以使用百度云盘下载：
链接：https://pan.baidu.com/s/1GlFnXiWYP0Q9tc059DvqAA 密码：t27x

## 1.2、 解压： ##
 [huangkai@sjq-10 local]$ **sudo tar -xvf kafka_2.12-2.0.0.tgz -C /usr/local/**

## 1.3、配置： ##

**sudo vim /usr/local/kafka_2.12-2.0.0/config/server.properties**

```
# brokerId，在集群中，此brokerId必须是唯一的
broker.id=0
	
# listeners
listeners=PLAINTEXT://192.168.64.128:9092

# 处理网络请求的线程数量
num.network.threads=3

# 用来处理磁盘IO的线程数量
num.io.threads=8

# 发送套接字的缓冲区大小
socket.send.buffer.bytes=102400

# 接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400

# 请求套接字的最大缓冲区大小
socket.request.max.bytes=104857600

#运行日志目录
log.dirs=/usr/local/kafka_2.12-2.0.0/logs

# topic在当前broker上的默认分区个数
num.partitions=1

# 用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1

offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

# segment文件保留的最长时间，超时将被删除
log.retention.hours=168

# segment文件大小，超出此数将创建新的文件
log.segment.bytes=1073741824

log.retention.check.interval.ms=300000

#zookeeper连接配置
zookeeper.connect=192.168.64.128:2181,192.168.64.129:2181,192.168.64.140:2181

# zookeeper连接超时配置
zookeeper.connection.timeout.ms=6000

#
group.initial.rebalance.delay.ms=0

是否可以删除topic,在 2.0版本中，默认值改为了true
delete.topic.enable=true
```

### 1.4、添加环境变量： ###

[huangkai@sjq-10 ~]$ **sudo vim + /etc/profile**
添加如下内容：
```
# KAFKA_HOME
export KAFKA_HOME=/usr/local/kafka_2.12-2.0.0
export PATH=:$KAFKA_HOME/bin:$PATH	
```
使配置文件生效：
[huangkai@sjq-10 ~]$ **source /etc/profile**

### 1.5、启动： ###

在启动 kafka 之前，需要先启动 zookeeper，关于 zookeeper的安装与启动，请点击 这里 查看。 
以守护进程方式：
 [huangkai@sjq-10 ~]$ **kafka-server-start.sh -daemon /usr/local/kafka_2.12-2.0.0/config/server.properties **

 ### 1.5、停止服务： ###

 [huangkai@sjq-10 ~]$ **kafka-server-stop.sh **


 ### 1.6、集群配置： ###
kafka 的集群配置非常简单，只需要将上面 1.3 中配置中的 broker.id 改为集群中全局唯一，  listeners 改为你安装的ip即可，假如上面配置的是 192.168.64.128，我需要在 192.168.64.129 上再配置一台加入 Kafka的集群中，129服务器的配置如下：

```
# brokerId，在集群中，此brokerId必须是唯一的
broker.id=1
	
# listeners
listeners=PLAINTEXT://192.168.64.129:9092

#运行日志目录
log.dirs=/usr/local/kafka_2.12-2.0.0/logs
```

其它配置几乎是相同的，除非你需要个性化配置

 ### 1.7、开机自动启动配置： ###
vim /usr/lib/systemd/system/kafka.service
添加内容如下：
```
[Unit]
Description=kafka
After=network.target
[Service]
Type=forking
User=huangkai
Group=huangkai
ExecStart=/usr/local/kafka_2.12-2.0.0/bin/kafka-server-start.sh -daemon /usr/local/kafka_2.12-2.0.0/config/server.properties
ExecReload=/usr/local/kafka_2.12-2.0.0/bin/kafka-server-restart.sh
ExecStop=/usr/local/kafka_2.12-2.0.0/bin/kafka-server-stop.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
 ### 1.8、kafka重启脚本： ###

[huangkai@sjq-01 ~]$ **vim /usr/local/kafka_2.12-2.0.0/bin/kafka-server-restart.sh**
```
#!/bin/bash

base_dir=$(dirname $0)

exec $base_dir/kafka-server-stop.sh

sleep 2

exec $base_dir/kafka-server-start.sh -daemon $base_dir/../config/server.properties
```
