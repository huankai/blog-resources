---
title: Kafka_02 Manager 安装与使用
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

# 一、概述： #

Kafka在雅虎内部被很多团队使用,媒体团队用它做实时分析流水线,可以处理高达20Gbps(压缩数据)的峰值带宽。
为了简化开发者和服务工程师维护Kafka集群的工作，构建了一个叫做Kafka管理器的基于Web工具，叫做 Kafka Manager。这个管理工具可以很容易地发现分布在集群中的哪些topic分布不均匀，或者是分区在整个集群分布不均匀的的情况。它支持管理多个集群、选择副本、副本重新分配以及创建Topic。同时，这个管理工具也是一个非常好的可以快速浏览这个集群的工具。
该软件是用Scala语言编写的。雅虎已经开源了Kafka Manager工具。这款Kafka集群管理工具主要支持以下几个功能：
1、管理几个不同的集群；
2、很容易地检查集群的状态(topics, brokers, 副本的分布, 分区的分布)；
3、选择副本；
4、产生分区分配(Generate partition assignments)基于集群的当前状态；
5、重新分配分区。

# 二、Kafka Manager下载及安装 #

项目地址：https://github.com/yahoo/kafka-manager
这个项目比 https://github.com/claudemamo/kafka-web-console 要好用一些，显示的信息更加丰富，kafka-manager本身可以是一个集群，kafka-manager没有权限管理功能。

下载：
[huangkai@sjq-01 install]$** wget https://github.com/yahoo/kafka-manager/archive/1.3.3.18.tar.gz**

也可以点击 如下百度云盘地址已编译好的安装包 ：
链接：https://pan.baidu.com/s/1b0SpFHrBrks-ujSXzjF9kg 密码：8w04

解压：
[huangkai@sjq-01 install]$ **tar -zxvf /data/install/kafka_manager_1.3.3.18.tar.gz -C /data/kafka_manager/**

解压后的是 kafka manager的源码，安装 kafka manager需要先安装 sbt，sbt详细安装文档可查看： https://www.scala-sbt.org/1.x/docs/Installing-sbt-on-Linux.html

sbt只需要在java运行环境，安装 sbt 只是为了编译kafka manager ，并不需要在每台服务器安装 sbt,安装 sbt 步骤如下：
```
[huangkai@sjq-01 install]$ curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
[huangkai@sjq-01 install]$ sudo yum install sbt

# 在第一次执行 sbt sbt-version 时，需要下载相应的jar包，可能需要一段时间，请耐心等待。
[huangkai@sjq-01 kafka-manager-1.3.3.18]$ sbt sbt-version
[info] Loading project definition from /data/kafka_manager/kafka-manager-1.3.3.18/project
Missing bintray credentials /home/huangkai/.bintray/.credentials. Some bintray features depend on this.
[info] Set current project to kafka-manager (in build file:/data/kafka_manager/kafka-manager-1.3.3.18/)
[info] 0.13.9
```

编译  kafka manager:
在编译的时候，也会下载相应的jar包，可能需要一段时间，编译完成后，会在当前目录下创建 target目录 ，编译后的文件为 target/universal/kafka-manager-1.3.3.18.zip 

```
[huangkai@sjq-01 kafka-manager-1.3.3.18]$ sbt clean dist

```

解压 编译后的包 kafka-manager-1.3.3.18.zip

```
[huangkai@sjq-01 kafka_manager]$ unzip kafka-manager-1.3.3.18.zip
```

启动 kafka manager:
[huangkai@sjq-01 kafka_manager]$ **nohup ./bin/kafka-manager -Dconfig.file=./conf/application.conf >/dev/null 2>&1 & **

启动之后，就可以使用浏览器访问 http://192.168.64.128:9000 ，默认端口号为 9000，可以使用 -Dhttp.port=9001 参数来指定端口号，如下图所示

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_manager_01.png)
