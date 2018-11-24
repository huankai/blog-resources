---
title: Hadoop 3.x 高可用配置
date: {{ date }}
author: huangkai
tags:
    - Hadoop3.x
---

# 一、高可用(HA)概述 #

- 所谓HA（high available），即高可用（7*24小时不中断服务）。
- 实现高可用最关键的策略是消除单点故障。HA严格来说应该分成各个组件的HA机制：HDFS的HA和YARN的HA。
- Hadoop2.0之前，在HDFS集群中NameNode存在单点故障（SPOF）。
- NameNode主要在以下两个方面影响HDFS集群
NameNode机器发生意外，如宕机，集群将无法使用，直到管理员重启
NameNode机器需要升级，包括软件、硬件升级，此时集群也将无法使用
HDFS HA功能通过配置Active/Standby两个nameNodes实现在集群中对NameNode的热备来解决上述问题。如果出现故障，如机器崩溃或机器需要升级维护，这时可通过此种方式将NameNode很快的切换到另外一台机器。

# 二、HDFS 高可用 #

## 2.1、HDFS 高可用概述 ##


## 2.2、HDFS 高可用配置 ##

### 2.2.1、高可用规划： ###

使用4台服务器，部署规划如下表：

|sjq128(192.168.64.128)|sjq129(192.168.64.129)|sjq130(192.168.64.130)|sjq150(192.168.64.150)|
|:---:|:---:|:---:|:---:|
|NameNode|<font color='red'>NameNode</font>|-|-|
|JournalNode|JournalNode|JournalNode|JournalNode|
|-|DataNode|DataNode|DataNode|
|Zookeeper|Zookeeper|Zookeeper|-|
|-|ResourceManager|-|-|
|-|NodeManager|NodeManager|NodeManager|

### 2.2.2、配置zookeeper ###

zookeeper 的集群安装请  [点击这里](https://www.baidu.com) 查看。

分别在 sjq128、sjq129、sjq130 启动 zookeeper:

```
[huangkai@sjq128 ~]$ /usr/local/zookeeper-3.4.9/bin/zkServer.sh start # 启动 zookeeper

[huangkai@sjq128 ~]$ /usr/local/zookeeper-3.4.9/bin/zkServer.sh status  # 查看zookeeper状态
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper-3.4.9/bin/../conf/zoo.cfg
Mode: follower
[huangkai@sjq128 ~]$ 
```

### 2.2.3、配置HDFS 高可用 ###

- 配置 `hadoop-env.sh`

```
export JAVA_HOME=/usr/java/default
```

- 配置 `core-site.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://mycluster</value>
	</property>
	<!-- 指定 Hadoop 运行时产生文件的存储目录，默认为 /tmp/hadoop-${user.name} -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/data/hadoop3.1.1/tmp</value>
	</property>
	
	<!-- hadoop中http访问的静态用户名，默认为 dr.who，没有什么特殊含义，也可以配置 dfs.permissions.enabled 为false 关闭权限检查 -->
	<property>
		<name>hadoop.http.staticuser.user</name>
		<value>huangkai</value>
	</property>
	<!-- 配置 dfs.permissions.enabled 为false 关闭权限检查,，生产环境不建议关闭，默认为true -->
	<!--
	<property>
		<name>dfs.permissions.enabled</name>
		<value>false</value>
	</property>
	 -->
	 
	 <!-- 配置zk  -->
	 <property>
		<name>ha.zookeeper.quorum</name>
		<value>sjq128:2181,sjq129:2181,sjq130:2181</value>
	</property>
</configuration>
```

- 配置 `hdfs-site.xml`

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
	<!-- 指定 HDFS副本备份数量，默认为3 -->
	<property>
		<name>dfs.replication</name>
		<value>2</value>
	</property>
	<!-- 指定 Hadoop secondaryNameNode 节点地址 -->
	<property>
		<name>dfs.namenode.secondary.http-address</name>
		<value>sjq129:9868</value>
	</property>
	<!-- 
	<property>
		<name>dfs.hosts</name>
		<value>/usr/local/hadoop-3.1.1/etc/hadoop/dfs.hosts</value>
	</property>
	<property>
		<name>dfs.hosts.exclude</name>
		<value>/usr/local/hadoop-3.1.1/etc/hadoop/dfs.hosts.exclude</value>
	</property>
	-->
	<!-- 完全分布式集群名称，要和core-site文件中的 fs.defaultFS名称一样 -->
	<property>
		<name>dfs.nameservices</name>
		<value>mycluster</value>
	</property>
	<!-- 集群中NameNode节点都有哪些 -->
	<property>
		<name>dfs.ha.namenodes.mycluster</name>
		<value>nn1,nn2</value>
	</property>
	<!-- nn1的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.mycluster.nn1</name>
		<value>sjq128:9000</value>
	</property>

	<!-- nn2的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.mycluster.nn2</name>
		<value>sjq129:9000</value>
	</property>
	<!-- nn1的http通信地址 -->
	<property>
		<name>dfs.namenode.http-address.mycluster.nn1</name>
		<value>sjq128:9870</value>
	</property>

	<!-- nn2的http通信地址 -->
	<property>
		<name>dfs.namenode.http-address.mycluster.nn2</name>
		<value>sjq129:9870</value>
	</property>
	
	<!-- 指定NameNode元数据在JournalNode上的存放位置 -->
	<property>
		<name>dfs.namenode.shared.edits.dir</name>
		<value>qjournal://sjq128:8485;sjq129:8485;sjq130:8485/mycluster</value>
	</property>
	<!-- 配置隔离机制，即同一时刻只能有一台服务器对外响应 -->
	<property>
		<name>dfs.ha.fencing.methods</name>
		<value>sshfence</value>
	</property>
	<!-- 使用隔离机制时需要ssh无秘钥登录-->
	<property>
		<name>dfs.ha.fencing.ssh.private-key-files</name>
		<value>/home/huankgai/.ssh/id_rsa</value>
	</property>
	<!-- 声明journalnode服务器存储目录-->
	<property>
		<name>dfs.journalnode.edits.dir</name>
		<value>/data/hadoop3.1.1/tmp/jn</value>
	</property>
		<!-- 访问代理类：client，mycluster，active配置失败自动切换实现方式-->
	<property>
  		<name>dfs.client.failover.proxy.provider.mycluster</name>
  		<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	
	<!-- 开启 HDFS-HA自动故障转移，默认为false -->
	<property>
		<name>dfs.ha.automatic-failover.enabled</name>
		<value>true</value>
	</property>
</configuration>

```
配置完后，将这些修改后的配置文件都同步到 hadoop的每个节点上。

- 启动 journalnode  服务
每台服务器执行：
```
[huangkai@sjq128 hadoop]$ hdfs --daemon start journalnode
```

- 在 nn1(sjq128) 服务器上对其进行格式化，并启动namenode
```
[huangkai@sjq128 hadoop]$ hdfs namenode -format
[huangkai@sjq128 hadoop]$ hdfs --daemon start namenode
```
- 在 nn2(sjq129) 上，同步nn1的元数据信息，并启动namenode
```
[huangkai@sjq129 hadoop]$ hdfs namenode -bootstrapStandby
[huangkai@sjq129 hadoop]$ hdfs --daemon start namenode
```

- 使用浏览器查看：
第一个namenode: http://sjq128:9870
第二个namenode: http://sjq129:9870

- 初始化HA在Zookeeper中状态
在 nn1 上执行:
```
[huangkai@sjq128 hadoop]$ hdfs zkfc -formatZK
```

- 启动HDFS服务
在 nn1 上执行:
```
[huangkai@sjq128 hadoop]$ start-dfs.sh
```

- 查看 nn1 与 nn2信息：
如下可知:nn1(sjq278) 为 standby ，nn2(sjq129) 为 active 状态
```
[huangkai@sjq128 hadoop]$ hdfs haadmin -getServiceState nn1
standby
[huangkai@sjq128 hadoop]$ hdfs haadmin -getServiceState nn2
active
[huangkai@sjq128 hadoop]$ 
```

- 检查
模拟 nn2(sjq129) namenode 进程宕机，查看 nn1(sjq128 )是否升级为 active.


# 三、YARN 高可用 #

