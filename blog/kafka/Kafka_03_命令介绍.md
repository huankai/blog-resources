---
title: Kafka_02 命令介绍
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

# 一、创建topic  #

```
--create : 创建topic 指令
--zookeeper : 指定 zookeeper地址
--replication-factor : 设置副本数

--partitions :设置分区数
[root@sjq-10 bin]# ./kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
# 二、查看topic信息  #