---
title: Kafka_02 命令介绍
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

# 一、topic  #

## 1.1、查看当前服务器所有topic: ##

```
[huangkai@sjq-01 config]$ kafka-topics.sh --zookeeper sjq-02:2181 --list
__consumer_offsets
streams_first
streams_second
test
test2
```
## 1.2、创建topic: ##

```
--create : 创建topic 指令
--zookeeper : 指定 zookeeper地址
--replication-factor : 设置副本数
--partitions :设置分区数
[root@sjq-10 bin]# ./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

## 1.3、删除 topic ##

如下，如果在 server.properties配置文件中配置了 delete.topic.enable 为true时，才会真正的删除 topic，否则，只会标记此topic为删除状态，不会真正的删除此topic。在1.0.0之后，此参数默认值为true
```
[huangkai@sjq-01 config]$ kafka-topics.sh --zookeeper sjq-02:2181 --delete --topic test
Topic test is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
[huangkai@sjq-01 kafka-manager-1.3.3.18]$
```

## 1.4、查看指定 topic 详情 ##

如下，该 topic 有3个 分区(partition) ，有一个副本数
```
[huangkai@sjq-01 config]$ kafka-topics.sh --zookeeper sjq-02:2181 --describe --topic test
Topic:test      PartitionCount:3        ReplicationFactor:1     Configs:
        Topic: test     Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: test     Partition: 1    Leader: 1       Replicas: 1     Isr: 1
        Topic: test     Partition: 2    Leader: 0       Replicas: 0     Isr: 0
[huangkai@sjq-01 config]$ 
```

# 二、测试 #

需求：测试同一个消费者组中的消费者，同一时刻只能有一个消费者消费

分别修改192.168.64.129，192.168.64.130 的${KAFKA_HOME}/config/consumer.properties文件 group.id 参数为任意相同值，也可不修改，默认都为 test-consumer-group

分别在 192.168.64.129，192.168.64.130 上启动kafka 消费者：


```
[huangkai@sjq-02  kafka_2.12-2.0.0]$ kafka-console-consumer.sh --bootstrap-server sjq-01:9092 --topic test -consumer.config ./config/consumer.properties
```

在 192.168.64.128 启动 kafka生产者：
```
[huangkai@sjq-01  kafka_2.12-2.0.0]$ kafka-console-producer.sh --broker-list sjq-01:9092 --topic test
```

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_08.png)

查看 在同一时刻，只用一个消费者收到了消息。