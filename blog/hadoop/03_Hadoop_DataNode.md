---
title: Hadoop - DataNode 介绍
date: {{ date }}
author: huangkai
tags:
    - Hadoop
---

DataNode负责将实际数据存储到HDFS中。

DataNode也称为 Slave。

NameNode 与 DataNode会不断保持心跳。

DataNode启动时，会将自己发布到NameNode并汇报负责自己持有的块列表。

当某个DataNode关闭时，它不会影响数据他集群的可用性，NameNode将安排由其他DataNode管理的块进行副本复制。

DataNode所在机器通常会配置有大的磁盘空间，因为实际数据存储在DataNode中。

DataNode会定期(**dfs.heartbeat.internal** 配置项，默认为3秒)向 NameNode发送心跳，如果NameNode长时间没有接收到DataNode的心跳，就会认为该DataNode失效。

block汇报时间间隔取参数值 **dfs.blockreport.internalMsec** ，默认时间为6小时。
