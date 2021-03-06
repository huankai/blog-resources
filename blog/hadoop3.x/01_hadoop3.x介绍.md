---
title: Hadoop 3.x 介绍
date: {{ date }}
author: huangkai
tags:
    - Hadoop3.x
---

# 一、Hadoop 3.x 新功能总结 #

## 1.1、JDK 版本： ##
hadoop 3.x 使用 jdk 8 运行编译的，最少需要 JDK 1.8

## 1.2、端口号调整 ##

<table><tr><td>分类</td><td>应用</td><td>Hadoop 2.x</td><td>Hadoop 3.x</td></tr><tr><td rowspan="3">NameNode(NN) ports</td><td >Namenode</td><td >8020</td><td >9820</td></tr><tr><td >NN HTTP UI</td><td >50070</td><td >9870</td></tr><tr><td >NN HTTPS UI</td><td >50470</td><td >9871</td></tr><tr><td rowspan="2">SecondaryNameNode（SNN） ports</td><td >SNN HTTP</td><td >50091</td><td >9869</td></tr><tr><td >SNN HTTP UI</td><td >50090</td><td >9868</td></tr><tr><td rowspan="4">DataNode(DN) ports</td><td >DN IPC</td><td >50020</td><td >9867</td></tr><tr><td >DN</td><td >50010</td><td >9866</td></tr><tr><td >DN HTTP UI</td><td >50075</td><td >9864</td></tr><tr><td >DN HTTPS UI</td><td >50475</td><td >9865</td></tr></table>


# 二、集群安装 #

部署环境说明：

|ip|主机名|服务器|
|:-:|:-:|:-:|
|192.168.117.150|hadoop150|NameNode、ResourceManager、NodeManager、DataNode|
|192.168.117.151|hadoop151|SecondaryNameNode、NodeManager、DataNode|
|192.168.117.152|hadoop152|NodeManager、DataNode|

安装JDK，jdk的安装方式请找度娘，这里略过。
配置 SSH，这里略过
下载 Hadoop : 

```
[huangkai@hadoop150 ~]$ wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.1.2/hadoop-3.1.2.tar.gz

```

解压到指定的目录: **tar -zxvf hadoop-3.1.2.tar.gz -C /opt/modules ** 

配置环境变量: **vim /etc/profile** ,在末尾添加如下内容:
```
HADOOP_HOME=/opt/modules/hadoop-3.1.2
#注意，hadoop的bin 和sbin都配置成环境变量，sbin就是super bin的简写，其实也是调用了bin目录中的相关脚本。
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

重新使配置文件生效: **source /etc/profile**

删除 ${HADOOP_HOME}/bin 、${HADOOP_HOME}/sbin 、${HADOOP_HOME}/etc/hadoop 目录下的 以 cmd 结尾的文件，这些文件是 windows 下文件，在 Linux 中不需要，此步为可选操作。

```
[huangkai@hadoop150 bin]$ rm -rf *.cmd
[huangkai@hadoop150 bin]$ cd ../sbin
[huangkai@hadoop150 sbin]$ rm -rf *.cmd
[huangkai@hadoop150 sbin]$ cd ../etc/hadoop
[huangkai@hadoop150 hadoop]$ rm -rf *.cmd
```

## 2.1、 hadoop-env.sh ##
该文件设置的是Hadoop运行时需要的环境变量，JAVA_HOME是必须设置的，即使当前系统环境变量中配置了JAVA_HOME,它也是不能识别的，因为Hadoop即使在本机上运行，它也会把当前的执行环境当成远程服务器。

```
# vim ${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
export JAVA_HOME= JAVA_HOME的绝对路径
```

## 2.2、core-site.xml ##
hadoop的核心配置文件，有默认的配置项目 core-default.xml
	
```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop150:9000</value>
    <description>指定 Hadoop 所使用的文件系统schema（URI），HDFS的老大(NameNode)的地址,协议可以是 hdfs://  tfs:// file:// gfs:// 等</description>
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/modules/hadoop-3.1.2/data/tmp</value>
	<description>指定 Hadoop 运行时产生文件的存储目录，默认为 /tmp/hadoop-${user.name}</description>
</property>
<property>
	<name>hadoop.http.staticuser.user</name>
	<value>huangkai</value>
	<description>hadoop中http访问的静态用户名，默认为 dr.who，没有什么特殊含义，也可以配置 dfs.permissions.enabled 为false 关闭权限检查，
	例如我是以 huangkai 用户启动的hadoop，没有配置此项，可能会导致在浏览器删除数据时出现无权限的错误，也可以使用 【hadoop fs -chmod 777 /] 修改文件权限</description>
</property>
 <!--
<property>
	<name>dfs.permissions.enabled</name>
	<value>false</value>
	<description>配置 dfs.permissions.enabled 为false 关闭权限检查,，生产环境不建议关闭，默认为true</description>
</property>
 -->
```

## 2.3、hdfs-site.xml ##
```
<property>
	<name>dfs.replication</name>
	<value>3</value>
	<description>指定 HDFS副本备份数量，默认为3</description>
</property>

<property>
	<name>dfs.namenode.secondary.http-address</name>
	<value>hadoop151:9868</value>
	<description>指定 Hadoop secondaryNameNode 节点地址</description>
</property>
```

## 2.4、mapred-site.xml ##

（在hadoop 2.x 版本中，并没有 mapred-site.xml 文件，而是 mapred-site.xml.template文件）修改配置内容：**vim mapred-site.xml**
```
<property>
    <name>mapreduce.frameworkname</name>
    <value>yarn</value>
	<description>指定 mapreduce 运行时的框架，这里指定  yarn ，默认是 local</description>
</property>
```

## 2.5、yarn-site.xml ##

```
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>hadoop150</value>
	<description>指定YARN 的老大 (ResourceManager)地址</description>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
	<description>NodeManager上运行的附属服务，需配置成 mapreduce_shuffle 才可以运行Mapreduce程序，默认值为 mapreduce.shuffle</description>
</property>
```

## 2.6、修改 workers  ##

在 hadoop 2.x版本中，修改的是 slaves 文件，在 3.x版本中，修改的是 workers 文件，workers 表示 hadoop的 slave，在此文件中添加hadoop的 slave主机名即可，每行代表一个slave。

```
hadoop150 # 如果配置了nameNode节点，也会在namenode节点上启动DataNode，根据需要配置
hadoop151
hadoop152

```

## 2.7 、每台服务器复制配置 ##

以上6个配置文件修改完成后，将 150上的 ${HADOOP_HOME}目录上传到 151 和 152 两台服务器上:
```
[root@hadoop150 modules]# yum -y install rsync
[root@hadoop150 modules]# rsync -av hadoop-3.1.2/ hadoop151:/opt/modules/hadoop-3.1.2
[root@hadoop150 modules]# rsync -av hadoop-3.1.2/ hadoop152:/opt/modules/hadoop-3.1.2
[root@hadoop150 modules]# chown huangkai:huangkai /opt/modules -R
```

分别在每台服务器上修改上传hadoop目录的所有者。

```
[root@hadoop150 local]# chown huangkai:huangkai /opt/modules -R  # 在 150 上执行
[root@hadoop151 local]# chown huangkai:huangkai /opt/modules -R  # 在 151 上执行
[root@hadoop152 local]# chown huangkai:huangkai /opt/modules -R  # 在 152 上执行
```


## 2.8、启动hadoop集群  ##
要启动Hadoop集群，需要启动HDFS 和 YARN两个集群。
注意：<font color='red'>**首次启动HDFS时，必须对其进行格式化操作。**</font>本质上是一些清理和准备工作，因为此时的HDFS在物理上还是不存在的。后续不再需要格式化。格式化的操作在HDFS集群的主角色（NameNode） 所在服务器上操作即可。

```
[huangkai@hadoop150 ~]# hdfs namenode -format # 或者hadoop namenode -format
```

### 2.8.1、单节点依次启动： ###
- 在主节点（192.168.117.150上使用以下命令启动HDFS NameNode:

```
[huangkai@hadoop150 hadoop-3.1.2]$ hdfs --daemon start namenode  # 或 hadoop-daemon.sh start namenode ，但已过时
```
- 在每个从节点(192.168.117.150,192.168.117.151,192.168.117.152)上使用以下命令启动HDFS DataNode:

```
[root@hadoop150 ~]# hdfs --daemon start datanode
```

- 在主节点上(192.168.117.150)使用以下命令启动YARN ResourceManager:

```
[root@hadoop150 ~]# yarn --daemon start resourcemanager
```

- 在每个从节点(192.168.117.150,192.168.117.151，192.168.117.152)上使用以下命令启动YARN nodemanager:

```
[root@hadoop150 ~]# yarn --daemon start nodemanager
```

以上脚本位于 ${HADOOP_HOME}/bin 目录下，如果想要停止某个角色，只需要把命令中的 `start` 改为 `stop` 即可。


### 2.8.2、SSH配置启动 ###
如果在 workers 文件中配置了 worker的主机名，并配置了 SSH 免密登陆，也可以使用如下方式启动：

以下脚本位于 ${HADOOP_HOME}/sbin 目录下

```
#只需要在 Namenode 所在节点执行：
[huangkai@hadoop150 hadoop-3.1.2]$ ./sbin/start-dfs.sh  # 会启动所有节点 namenode、dataNodes、secondaryNameNodes 服务
Starting namenodes on [hadoop150]
Starting datanodes
hadoop151: WARNING: /opt/modules/hadoop-3.1.2/logs does not exist. Creating.
hadoop152: WARNING: /opt/modules/hadoop-3.1.2/logs does not exist. Creating.
Starting secondary namenodes [hadoop151]
[huangkai@hadoop150 hadoop-3.1.2]$ ./sbin/start-yarn.sh  # 会启动所有节点 resourcemanager、nodemanagers 服务
Starting resourcemanager
Starting nodemanagers
[huangkai@hadoop150 hadoop-3.1.2]$ 
```

停止

```
#只需要在 Namenode 所在节点执行：
[root@sjq128 ~]# stop-yarn.sh #会停止所有节点 namenode、dataNodes、secondaryNameNodes 服务
[huangkai@sjq128 hadoop-3.1.1]$ stop-dfs.sh  # 会停止所有点点 resourcemanager、nodemanagers 服务

```

### 2.8.3、浏览器查看 ###

集群启动成功后，可以使用 9870(在Hadoop 2.x版本中端口为 50070) 端口访问 NameNode：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/hadoop-3-x/hadoop_01.png)

使用 8088 端口访问 ResourceManager：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/hadoop-3-x/hadoop_02.png)


使用  9864(在Hadoop 2.x版本中端口为 50075) 端口访问 DataNode（每台部署DataNode的都可以访问）：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/hadoop-3-x/hadoop_03.png)


使用  9868(在Hadoop 2.x版本中端口为 50090) 端口访问 secondaryNameNode：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/hadoop-3-x/hadoop_04.png)

# 三、服役新节点 #

随着公司业务的增长，数据量越来越大的时候，原有的数据节点容量已不能满足存储数据的需求，需要在原有的集群基础上动态添加新的数据节点。

## 3.1、环境准备 ##
- 新节点IP: 192.168.117.153，主机名：hadoop153
- 每台节点上配置 hosts
```
[huangkai@hadoop150 hadoop]$ sudo vim /etc/hosts
192.168.117.150  hadoop150
192.168.117.151  hadoop151
192.168.117.152  hadoop152
192.168.117.153  hadoop153
```
- 在NameNode节点上配置新节点免密登陆：
```
[huangkai@hadoop150 local]$ ssh-copy-id -i ~/.ssh/id_rsa.pub 192.168.117.153
```
- 将namenode 的 hadoop 复制到新节点指定的目录：
```
[huangkai@hadoop150 local]$ rsync -av hadoop-3.1.2/ hadoop153:/opt/modules/hadoop-3.1.2/
```
- 修改新节点文件权限，并删除日志目录，创建数据目录、配置环境变量：
```
[huangkai@hadoop153 local]$ sudo chown huangkai:huangkai hadoop-3.1.2/ -R
[huangkai@hadoop153 hadoop-3.1.2]$ rm -rf data logs/*  # 注意：从 nameNode 节点复制过来的 data 目录一定要删除，如果不删除，在 data/tmp/dfs/data/current/VERSION 文件中的 datanodeUuid 会重复，会将两个节点当作同一台服务器处理。
[huangkai@hadoop153 hadoop-3.1.2]$ sudo chown huangkai:huangkai /data/ -R
[huangkai@hadoop153 hadoop-3.1.2]$ sudo vim /etc/profile	
	# hadoop 
	HADOOP_HOME=/usr/local/hadoop-3.1.2
	export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
[huangkai@hadoop153 hadoop-3.1.2]$ source /etc/profile
```

## 3.2、创建 dfs.host文件 ##

在nameNode 节点上的 ${HADOOP_HOME}/etc/hadoop/ 目录下创建 dfs.hosts 文件，文件内容如下(包含新服务的节点)：

```
[huangkai@hadoop150 hadoop]$ vim dfs.hosts 
hadoop150
hadoop151
hadoop152
hadoop153
[huangkai@hadoop150 hadoop]$
```

## 3.3、配置 hdfs-site.xml 文件 ##

在nameNode 节点上修改 hdfs-site.xml 文件，添加如下内容：

```
...
<!-- 集群添加节点配置 -->
<property>
	<name>dfs.hosts</name>
	<value>/opt/modules/hadoop-3.1.2/etc/hadoop/dfs.hosts</value>
</property>
....
```
## 3.4、刷新 namenode ##
在nameNode 节点执行:
```
[huangkai@hadoop150 hadoop]$ ../../bin/hdfs dfsadmin -refreshNodes
Refresh nodes successful
[huangkai@hadoop150 hadoop]$ 
```
## 3.5、更新 resourcemanager ##
在nameNode 节点执行:
```
[huangkai@hadoop150 hadoop]$ ../../bin/yarn rmadmin -refreshNodes
2019-06-24 19:23:40,448 INFO client.RMProxy: Connecting to ResourceManager at hadoop150/192.168.117.150:8033
[huangkai@hadoop150 hadoop]$ ps aux|grep java
```
## 3.6、单独命令启动新的数据节点和节点管理器 ##

```
[huangkai@hadoop153 hadoop-3.1.2]$ ./bin/hdfs --daemon start datanode # 启动datanode
[huangkai@hadoop153 hadoop-3.1.2]$ ./bin/yarn --daemon start nodemanager #启动 nodemanager
[huangkai@hadoop153 hadoop]$ ps aux|grep java # 查看进程
[huangkai@sjq140 install]$ 
```

## 3.7、浏览器查看 ##
如下，新的节点已成功添加：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/hadoop-3-x/hadoop_05.png)

## 3.8、数据均匀负载 ##
如果数据不均匀，可以使用如下命令实现集群的再平衡，在namenode节点上执行：

```
[huangkai@sjq128 hadoop]$ start-balancer.sh 
[huangkai@sjq128 hadoop]$
```

# 四、	退役旧数据节点 #

## 4.1、创建dfs.hosts.exclude 文件 ##
在namenode节点的 ${HADOOP_HOME}/etc/hadoop目录下创建 dfs.hosts.exclude 文件

```
[huangkai@sjq128 hadoop]$ vim dfs.hosts.exclude #在文件中输入要排除的节点 hostname
sjq140
```

## 4.2、配置namenode节点 hdfs-site.xml 文件 ##

```
<!-- 集群排除节点配置 -->
<property>
	<name>dfs.hosts.exclude</name>
	<value>/usr/local/hadoop-3.1.1/etc/hadoop/dfs.hosts.exclude</value>
</property>
```

## 4.3、刷新 namenode、刷新resourcemanager ##

```
[huangkai@sjq128 hadoop]$ hdfs dfsadmin -refreshNodes
Refresh nodes successful
[huangkai@sjq128 hadoop]$ yarn rmadmin -refreshNodes
2018-11-21 17:27:29,725 INFO client.RMProxy: Connecting to ResourceManager at sjq128/192.168.64.128:8033
[huangkai@sjq128 hadoop]$ 
```

## 4.4、使用浏览器查看（3.1.1版本从这步开始好像不行了，节点并没有退役，待完善...） ##


## 4.5、删除退役节点 ##

1、从 namenode节点的 dfs.hosts 文件中删除退役节点的 Hostname

```
[huangkai@sjq128 hadoop]$ vim dfs.hosts # 将 退役的节点 hostname 删除，保存
```
2、刷新nameNode，刷新resourcemanager
```
[huangkai@sjq128 hadoop]$ hdfs dfsadmin -refreshNodes
Refresh nodes successful
[huangkai@sjq128 hadoop]$ yarn rmadmin -refreshNodes
2018-11-21 17:37:27,129 INFO client.RMProxy: Connecting to ResourceManager at sjq128/192.168.64.128:8033
[huangkai@sjq128 hadoop]$ 
```

## 4.6、删除 workers 文件中配置的 退役节点的 Hostname ##

```
[huangkai@sjq128 hadoop]$ vim workers # 将 退役的节点 hostname 删除，保存
```

## 4.7、单独停止退役节点的DataNode 进程  ##

在退役节点的服务器执行（如这里退役是节点是 sjq140）
```
[huangkai@sjq140 hadoop]$ hdfs --daemon stop datanode
[huangkai@sjq140 hadoop]$ yarn --daemon stop nodemanager
[huangkai@sjq140 hadoop]$ jps 
17862 Jps
[huangkai@sjq140 install]$ 
```

## 4.8、重新负载均衡(可选) ##

如果数据不均匀，可以使用如下命令实现集群的再平衡，在namenode节点上执行：

```
[huangkai@sjq128 hadoop]$ start-balancer.sh 
[huangkai@sjq128 hadoop]$
```