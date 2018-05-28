---
title: Hadoop简介与安装
date: {{ date }}
author: huangkai
tags:
    - Hadoop
---

# 一、Hadoop介绍 #
Hadoop 是Apache 旗下用java语言实现的开源软件框架，是一个开发和运行处理大规模数据的软件平台，允许使用简单的编程模型在大量的计算机集群上对大型数据进行分布式处理。
	
Hadoop目前有1.x / 2.x / 3.x 三个大版本：
- 1.x : 主要由 HDFS(分布式文件系统（redundant reliable storage）) 和 MapReduce(分布式运算变成框架(cluster resource management & data processing)) 组件构成。
- 2.x : 主要有 HDFS 、YARN(作业调度和集群资源管理框架) 、 MapReduce Or  Others 组件构成。在1.x版本中，MapReduce主要是集群资源管理，在2.x版本中YARN做集群资源管理和作业调度，MapReduce只用于数据处理，也可以使用其它的一些技术做数据管理。
- 3.x : 刚出来的新版本。
	
Hadoop是 Apache Lucene 创始人Dou Cutting 创建的，最早起源于Nutch,它是Lucene(全文检索框架)的子项目，Nutch的设计目标是构建一个大型的全网搜索引擎，包括网页抓取、索引、查询等功能。但随着抓取网页数量的增加，遇到了严重的可扩展性问题，如何解决数十亿网页的存储问题和索引问题。
2003年Google发表了一篇论文为该问题提供了可行的解决方案，论文中描述的是谷歌的产品架构，该架构称为：**谷歌分布式文件系统(GFS)**,可以解决在网页爬取和索引过程中产生的超大文件的存储需求。
2004年,Google发表论文向全世界介绍了谷歌版的**MapReduce**系统。
同时期，Nutch的开发人员完成了相应的开源实现HDFS和 MapReduce，并从Nutch中剥离出来成为独立的Hadoop项目，到2008年1月，Hadoop成为Apache顶级项目，迎来了它的快速发展期。
2006年Google发表了论文是关于**BigTable**的，这促使了后来的Hbase的发展。
因此，Hadoop极其生态圈的发展离不开Google的贡献。
	
# 	二、Hadoop的特性  #

- 扩容能力：Hadoop是在可用的计算机集群之间分配数据并完成计算任务的，这些集群可以方便的扩展到数以千计的节点中； 
- 成本低：Hadoop通过普通廉价的机器组成服务器集群来分发以及处理数据，以至于成本很低； 
- 高效率：通过并发数据，Hadoop可以在节点之间动态并行的移动数据，使得速度非常快；
- 可靠性：能自动维护数据的多份复制，并且在任务失败后能自动重新部署(redeploy) 计算任务，所以Hadoop的按位存储和处理数据的能力值得信赖。

# 	三、集群安装  #

集群介绍：
Hadoop集群具体来说包括两个集群：HDFS集群和YARN集群，两者逻辑上分离(HDFS的运行不影响YARN,YARN的运行也不影响HDFS)，但物理上常部署在一起(HDFS和YARN可以部署在同一服务器上)。
- HDFS 集群负责海量数据的存储，集群中的角色主要有：NameNode、DataNode、SecondaryNameNode(相当于NameNode的秘书)
- YARN集群负责海量数据运算时的资源调度，集群中的角色主要有：ResourceManager、NodeManager
- Mapreduce 其实是一个分布式运算编程的框架，是应用程序开发包，由用户按照编程规范进行程序开发，打包运行在HDFS集群上并且受到YARN集群的资源调度管理。

Hadoop 常用的集群方式有三种:
- Standlone mode(独立模式),只在单机部署，仅一台机器运行一个java进程 ，主要用于调试；
- Pseude-Distributed mode(伪分布模式)，只在单机部署，一台机器运行HDFS的NameNode、DataNode、YARN的ResourceManager、NodeManager,但分别启用单独的java进程 ，主要用于调试；
- Cluster mode(集群模式):生产环境部署，会使用N 台主机组成一个Hadoop集群，主节点与从节点会分开部署在不同服务器上。

## 3.1、安装前配置 ##
操作系统: **CentOS Linux release 7.3.1611 (Core) **
JDK版本: ** 1.8.0_144 **
Hadoop版本: **hadoop-2.9.1**
关闭防火墙: **systemctl stop firewalld**
设置防火墙开机禁止启动: **systemctl disable firewalld**
配置主机名(三台机器都需要配置)： **vim /etc/hosts**
```
192.168.64.128  hadoop-node-1
192.168.64.129  hadoop-node-2
192.168.64.130  hadoop-node-3
```
重启网卡: **systemctl restart network**
使用 ping 命令检查是否配置成功(三台机器都需要配置成功)，这一步必须不能出错
```
[root@sjq-02 ~]# ping -c 3 hadoop-node-1
PING hadoop-node-1 (192.168.64.128) 56(84) bytes of data.
64 bytes from brokerserver1 (192.168.64.128): icmp_seq=1 ttl=64 time=0.523 ms
64 bytes from brokerserver1 (192.168.64.128): icmp_seq=2 ttl=64 time=1.71 ms
64 bytes from brokerserver1 (192.168.64.128): icmp_seq=3 ttl=64 time=1.69 ms

--- hadoop-node-1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.523/1.309/1.713/0.555 ms
[root@sjq-02 ~]# 
[root@sjq-02 ~]# ping -c 3 hadoop-node-2
PING hadoop-node-2 (192.168.64.129) 56(84) bytes of data.
64 bytes from brokerserver2 (192.168.64.129): icmp_seq=1 ttl=64 time=0.013 ms
64 bytes from brokerserver2 (192.168.64.129): icmp_seq=2 ttl=64 time=0.024 ms
64 bytes from brokerserver2 (192.168.64.129): icmp_seq=3 ttl=64 time=0.024 ms

--- hadoop-node-2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.013/0.020/0.024/0.006 ms
[root@sjq-02 ~]# 
[root@sjq-02 ~]# ping -c 3 hadoop-node-3
PING hadoop-node-3 (192.168.64.130) 56(84) bytes of data.
64 bytes from hadoop-node-3 (192.168.64.130): icmp_seq=1 ttl=64 time=0.846 ms
64 bytes from hadoop-node-3 (192.168.64.130): icmp_seq=2 ttl=64 time=1.36 ms
64 bytes from hadoop-node-3 (192.168.64.130): icmp_seq=3 ttl=64 time=0.975 ms

--- hadoop-node-3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.846/1.060/1.360/0.219 ms
[root@sjq-02 ~]# 
```

配置免密登陆(128可以使用 **ssh** 免密登陆129和130两台服务器):
在128服务器上执行如下：
```
#第一步：生成公钥私钥
[root@sjq-01 ~]# ssh-keygen -t rsa
#输入以上命令，按下三个回车。

#第二步：按锁头
#命令语法为: ssh-copy-id -i ~/.ssh/id_rsa.pub <romte_ip>
#如果你的ssh端口不是默认的22,可以使用 -p 参数指定端口号,如:（ssh-copy-id -i ~/.ssh/id_rsa.pub -p 24 <romte_ip>）
[root@sjq-01 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.64.129

#回车输入密码即可使用 (ssh 192.168.64.129) 登陆到129服务器，
#配置免密登陆130服务器配置也是如此。
```

部署环境:

|主机|IP|角色分配|
|:--:|:--:|:--:|
|hadoop-node-1|192.168.64.128|NameNode、DataNode、ResourceManager|
|hadoop-node-2|192.168.64.129|DataNode、NodeManager、SecondaryNameNode|
|hadoop-node-3|192.168.64.130|DataNode、NodeManager|
## 3.2、安装JDK 1.8 ##
安装方式网上一大把，这里略过。

## 3.3、安装Hadoop ##

下载 Hadoop : [http://hadoop.apache.org/releases.html#Download]()

解压到指定的目录: **tar -zxvf hadoop-2.9.1.tar.gz -C /usr/local/** 

配置环境变量: **vim /etc/profile** ,在末尾添加如下内容:
```
HADOOP_HOME=/usr/local/hadoop-2.9.1
#注意，hadoop的bin 和sbin都配置成环境变量，sbin就是super bin的简写，其实也是调用了bin目录中的相关脚本。
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

重新使配置文件生效: **source /etc/profile**
	
# 	四、Hadoop配置文件修改(每台机器执行) #

## 3.1、 hadoop-env.sh ##
该文件设置的是Hadoop运行时需要的环境变量，JAVA_HOME是必须设置的，即使当前系统环境变量中配置了JAVA_HOME,它也是不能识别的，因为Hadoop即使在本机上运行，它也会把当前的执行环境当成远程服务器。
```
vim ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
	
export JAVA_HOME= JAVA_HOME的绝对路径.
```	

## 3.2、core-site.xml ##
hadoop的核心配置文件，有默认的配置项目 core-default.xml
	
```
<!-- 指定 Hadoop 所使用的文件系统schema（URI），HDFS的老大(NameNode)的地址,协议可以是 hdfs://  tfs:// file:// gfs:// 等 -->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop-node-1:9000</value>
</property>
<!-- 指定 Hadoop 运行时产生文件的存储目录，默认为 /tmp/hadoop-${user.name} -->
<property>
    <name>hadoop.tmp.dir</name>
    <value>/data/hadoop/tmp</value>
</property>
```

## 3.3、hdfs-site.xml ##
```
<!-- 指定 HDFS副本备份数量，默认为3 -->
<property>
    <name>dfs.replication</name>
    <value>2</value>
</property>

<!-- 指定 Hadoop secondaryNameNode 节点地址 -->
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>hadoop-node-2:50090</value>
</property>
```

## 3.4、mapred-site.xml ##
进入 ${HADOOP_HOME}/etc/hadoop目录下，你会发现并没有 mapred-site.xml文件，而是有一个mapred-site.xml.template文件，这是 mapred-site.xml 的模板文件，使用 mv 命令重命名:
```
[root@sjq-01 hadoop]# mv mapred-site.xml.template mapred-site.xml
```

修改配置内容：**vim mapred-site.xml**
```
<!-- 指定 mapreduce 运行时的框架，这里指定  yarn ，默认是 local -->
<property>
    <name>mapreduce.frameworkname</name>
    <value>yarn</value>
</property>

```

## 3.5、yarn-site.xml ##

```
<!-- 指定YARN 的老大 (ResourceManager)地址 -->
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop-node-1</value>
</property>
<!-- NodeManager上运行的附属服务，需配置成 mapreduce_shuffle 才可以运行Mapreduce程序，默认值为 mapreduce.shuffle -->
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

## 3.6、slaves  ##

此文件写上从节点的主机名，每一行写一个主机名
```
hadoop-node-1
hadoop-node-2
hadoop-node-3

```

## 3.7、关于Hadoop 的配置文件 ##
以上我们主要修改的都 `***-site.xml`配置文件，在Hadoop中，有这些默认的只读的配置文件， core-default.xml, hdfs-default.xml, yarn-default.xml and mapred-default.xm 配置文件，这里面配置了hadoop的默认配置选项，如果用户没有修改，这些文件中的配置将会生效；

`***-site.xml` 这些配置了用户自定义的配置选项
site 配置选项的优先级会大于 default 文件中的配置，如果有配置的话，就会覆盖 `***-default.xml` 中的配置选项。

所有的配置文件选项，可以在官网文档中查询:
core-default.xml ： http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/core-default.xml
hdfs-default.xml ：http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml
mapred-default.xml ： http://hadoop.apache.org/docs/r2.9.1/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml
yarn-default.xml ： http://hadoop.apache.org/docs/r2.9.1/hadoop-yarn/hadoop-yarn-common/yarn-default.xml
与上个版本相比过时的配置 ： http://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/DeprecatedProperties.html
## 3.8 、每台服务器复制配置 ##

以上6个配置文件修改完成后，将 128上的 ${HADOOP_HOME}目录上传到 129 和 130两台服务器上:
```
[root@sjq-02 local]# scp -r /usr/local/hadoop-2.9.1 root@192.168.64.129:/usr/local
[root@sjq-02 local]# scp -r /usr/local/hadoop-2.9.1 root@192.168.64.130:/usr/local
```

修改 129 和 130 两台服务器上的环境变量(分别在129 和 130上执行以下命令):
```
[root@sjq-02 ~]# vim /etc/profile
# 添加如下内容
HADOOP_HOME=/usr/local/hadoop-2.9.1
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

[root@sjq-02 ~]# source /etc/profile
```

# 五、Hadoop 集群启动 #

## 5.1、启动方式 ##
要启动Hadoop集群，需要启动HDFS 和 YARN两个集群。
注意：<font color='red'>**首次启动HDFS时，必须对其进行格式化操作。**</font>本质上是一些清理和准备工作，因为此时的HDFS在物理上还是不存在的。后续不再需要格式化。格式化的操作在HDFS集群的主角色（namenode） 所在服务器上操作即可。

```
[root@sjq-01 ~]# hdfs namenode -format # 或者hadoop namenode -format
```

单节点依次启动：
- 在主节点上使用以下命令启动HDFS NameNode:

```
[root@sjq-01 ~]# hadoop-daemon.sh start namenode
```
- 在每个从节点上使用以下命令启动HDFS DataNode:

```
[root@sjq-01 ~]# hadoop-daemon.sh start datanode
```

- 在主节点上使用以下命令启动YARN ResourceManager:

```
[root@sjq-01 ~]# yarn-daemon.sh start resourcemanager
```

- 在每个从节点上使用以下命令启动YARN nodemanager:

```
[root@sjq-01 ~]# yarn-daemon.sh start nodemanager
```

以上脚本位于 ${HADOOP_HOME}/sbin 目录 下，如果想要停止某个角色，只需要把命令中的 `start` 改为 `stop` 即可。