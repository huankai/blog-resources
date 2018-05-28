---
title: Hadoop - HDFS工作机制
date: {{ date }}
author: huangkai
tags:
    - Hadoop
---

NameNode负责管理整个文件系统元数据，DataNode负责具体的文件数据块存储，Secondary NameNode 主要协助NameNode进行元数据的备份。

HDFS的内部工作机制对客户端都是透明的，客户端请求访问HDFS 都需要通过NameNode申请来访问。