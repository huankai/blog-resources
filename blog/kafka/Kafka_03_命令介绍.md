---
title: Kafka_03 命令介绍
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

# 一、topic  #

## 1.1、查看当前服务器所有topic: ##

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-topics.sh --zookeeper sjq-02:2181 --list
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
[root@sjq-10 kafka_2.12-2.0.0]# ./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic test
```

## 1.3、删除 topic ##

如下，如果在 server.properties配置文件中配置了 delete.topic.enable 为true时，才会真正的删除 topic，否则，只会标记此topic为删除状态，不会真正的删除此topic。在1.0.0之后，此参数默认值为true
```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-topics.sh --zookeeper sjq-02:2181 --delete --topic test
Topic test is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
[huangkai@sjq-01 kafka-manager-1.3.3.18]$
```

## 1.4、查看指定 topic 详情 ##

如下，该 topic 有3个 分区(partition) ，有一个副本数
```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-topics.sh --zookeeper sjq-02:2181 --describe --topic test
Topic:test      PartitionCount:3        ReplicationFactor:1     Configs:
        Topic: test     Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: test     Partition: 1    Leader: 1       Replicas: 1     Isr: 1
        Topic: test     Partition: 2    Leader: 0       Replicas: 0     Isr: 0
[huangkai@sjq-01 config]$ 
```

## 1.5、修改topic信息 ##

修改 topic 为 test的 partition 数：

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-topics.sh --zookeeper sjq-01:2181 --alter --topic test --partitions 4
```

再次查看test 主题信息：

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-topics.sh --zookeeper sjq-02:2181 --describe --topic test            
Topic:test      PartitionCount:4        ReplicationFactor:1     Configs:
        Topic: test     Partition: 0    Leader: 2       Replicas: 2     Isr: 2
        Topic: test     Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: test     Partition: 2    Leader: 1       Replicas: 1     Isr: 1
        Topic: test     Partition: 3    Leader: 2       Replicas: 2     Isr: 2
[huangkai@sjq-01 kafka_2.12-2.0.0]$
```

## 1.5、kafka集群自动分区平衡 ##
在创建 topic时，kafka会尽量的将partition均匀的分布在所有的 Brokers 上，并将replicas（partition副本） 也均匀的分页在所有的 brokers上，每个partition的所有replicas 叫做 "assigned replicas"， assigned replicas中第一个 replicas叫 "preferred replicas"，刚创建的preferred replicas是 leader，负责所有读写。
但随着时间的推移，broker可能会宕机、导致 leader迁移，机器的负载不均匀，此时，我们期望 对 topic 的 leader进行重新的负载。

- 对所有topic 进行操作
	```
	[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-preferred-replica-election.sh --zookeeper sjq-01:2181,sjq-02:2181,sjq-03:2181
	```

- 对指定 topic进行操作：

	```
	[huangkai@sjq-01 kafka_2.12-2.0.0]$ vim topicPartitionList.json #创建要分区平衡的topic json文件，格式如下
	{
	    "partitions": [
	        {
	            "topic": "test",
	            "partition": "0"
	        }
	    ]
	}
	
	[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-preferred-replica-election.sh --zookeeper sjq-01:2181,sjq-02:2181,sjq-03:2181 --path-to-json-file topicPartitionList.json
	```

以上两种方式是手动执行命令，还可以根据配置参数来开启自动分区平衡

在 ${KAFKA_HOME}/config/server.properties配置文件中配置如下参数，默认为也为 true。
```
auto.leader.rebalance.enable=true
```


## 1.6、kafka集群分区日志迁移 ##

案例：如下，test 主题有 3 个 partition,partition 0 在 broker 0上，partition 1 在 broker 1上，partition 2 在 broker 2上，现在将 test的 partition迁移到 broker 1 和  borker 2 上。

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-topics.sh --zookeeper sjq-02:2181 --describe --topic test                                         
Topic:test      PartitionCount:3        ReplicationFactor:1     Configs:
        Topic: test     Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: test     Partition: 1    Leader: 1       Replicas: 1     Isr: 1
        Topic: test     Partition: 2    Leader: 2       Replicas: 2     Isr: 2
[huangkai@sjq-01 kafka_2.12-2.0.0]$
```

### 1.6.1、生成迁移计划 ###

创建迁移计划文件：

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ vim topics-to-move.json #内容如下，指定需要迁移的topic，可以指定多个
{
    "topics": [
        {
            "topic": "test"
        }
    ],
    "version": 1
}

```

使用 `-generate` 生成迁移计划(这一步只是生成迁移计划，并没有执行数据迁移):
```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-reassign-partitions.sh --zookeeper sjq-01:2181 --topics-to-move-json-file topics-to-move.json --broker-list "1,2" --generate
Current partition replica assignment
{"version":1,"partitions":[{"topic":"test","partition":2,"replicas":[2],"log_dirs":["any"]},{"topic":"test","partition":1,"replicas":[1],"log_dirs":["any"]},{"topic":"test","partition":0,"replicas":[0],"log_dirs":["any"]}]}

Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"test","partition":1,"replicas":[2],"log_dirs":["any"]},{"topic":"test","partition":0,"replicas":[1],"log_dirs":["any"]},{"topic":"test","partition":2,"replicas":[1],"log_dirs":["any"]}]}
[huangkai@sjq-01 kafka_2.12-2.0.0]$
```

### 1.6.2、执行计划 ###
上一步生成了一些信息，将下面的信息写入到一个json文件中，如下：

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ vim reassignment-node.json
{"version":1,"partitions":[{"topic":"test","partition":1,"replicas":[2],"log_dirs":["any"]},{"topic":"test","partition":0,"replicas":[1],"log_dirs":["any"]},{"topic":"test","partition":2,"replicas":[1],"log_dirs":["any"]}]}

```

使用 `-execute` 执行计划:

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-reassign-partitions.sh --zookeeper sjq-01:2181 --reassignment-json-file reassignment-node.json --execute          
Current partition replica assignment

{"version":1,"partitions":[{"topic":"test","partition":2,"replicas":[2],"log_dirs":["any"]},{"topic":"test","partition":1,"replicas":[1],"log_dirs":["any"]},{"topic":"test","partition":0,"replicas":[0],"log_dirs":["any"]}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions.
[huangkai@sjq-01 kafka_2.12-2.0.0]$
```

查看运行结果：

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$  kafka-reassign-partitions.sh --zookeeper sjq-01:2181 --reassignment-json-file reassignment-node.json --verify
Status of partition reassignment: 
Reassignment of partition test-1 completed successfully
Reassignment of partition test-0 completed successfully
Reassignment of partition test-2 completed successfully
[huangkai@sjq-01 kafka_2.12-2.0.0]$ 
```

再次查看 tes主题信息，如下(partition 0 已分配到 broker 1上，现在 test主题只在 borker 1 与 broker 2 上存在了)：

```
[huangkai@sjq-01 kafka_2.12-2.0.0]$ kafka-topics.sh --zookeeper sjq-02:2181 --describe --topic test
Topic:test      PartitionCount:3        ReplicationFactor:1     Configs:
        Topic: test     Partition: 0    Leader: 1       Replicas: 1     Isr: 1
        Topic: test     Partition: 1    Leader: 2       Replicas: 2     Isr: 2
        Topic: test     Partition: 2    Leader: 1       Replicas: 1     Isr: 1
[huangkai@sjq-01 kafka_2.12-2.0.0]$
```

