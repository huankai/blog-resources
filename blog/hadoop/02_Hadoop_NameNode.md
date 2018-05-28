---
title: Hadoop - NameNode 介绍
date: {{ date }}
author: huangkai
tags:
    - Hadoop
---

NameNode 是Hadoop 的核心。 

NameNode 也称为 Master。

NameNode仅存储HDFS的元数据、文件系统中所有的目录树、并跟踪整个集群中的文件。

NameNode不存储实际的数据或数据集，数据本身是存储在每个DataNode节点中。

NameNode知道HDFS中任何给定文件列表的块的列表及其位置，使用此信息，NameNode知道如何从块中构建文件。

NameNode并不会持久化存储每个文件中各个块所在DataNode的位置信息，这些信息会在系统启动时从数据节点重建。

NameNode对HDFS至关重要，当NameNode节点无法使用时，HDFS/Hadoop整个集群都无法使用。

NameNode所在机器通常会配置大内存。 