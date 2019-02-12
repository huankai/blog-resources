---
title: 04_RabbitMQ 集群
date: {{ date }}
author: huangkai
tags:
    - RabbitMQ
---


Rabbitmq集群大概分为二种方式：
（1）普通模式：默认的集群模式。
（2）镜像模式：把需要的队列做成镜像队列。
一个rabbitmq集 群中可以共享 user，vhost，queue，exchange等，所有的数据和状态都是必须在所有节点上复制的，一个例外是，那些当前只属于创建它的节点的消息队列，尽管它们可见且可被所有节点读取。rabbitmq节点可以动态的加入到集群中，一个节点它可以加入到集群中，也可以从集群环境中移除。
   集群中有两种节点：
1 内存节点：只保存状态到内存（一个例外的情况是：持久的queue的持久内容将被保存到disk）
2 磁盘节点：保存状态到内存和磁盘。
内存节点虽然不写入磁盘，但是它执行比磁盘节点要好。集群中，只需要一个磁盘节点来保存状态 就足够了如果集群中只有内存节点，那么不能停止它们，否则所有的状态，消息等都会丢失。

良好的设计架构可以如下：在一个集群里，有3台以上机器，其中1台使用磁盘模式，其它使用内存模式。其它几台为内存模式的节点，无疑速度更快，因此客户端（consumer、producer）连接访问它们。而磁盘模式的节点，由于磁盘IO相对较慢，因此仅作数据备份使用。

# 一、普通模式 #

默认的集群模式，queue创建之后，如果没有其它policy，则queue就会按照普通模式集群。对于Queue来说，消息实体只存在于其中一个节点，A、B两个节点仅有相同的元数据，即队列结构，但队列的元数据仅保存有一份，即创建该队列的rabbitmq节点（A节点），当A节点宕机，你可以去其B节点查看，./rabbitmqctl list_queues 发现该队列已经丢失，但声明的exchange还存在。当消息进入A节点的Queue中后，consumer从B节点拉取时，RabbitMQ会临时在A、B间进行消息传输，把A中的消息实体取出并经过B发送给consumer，所以consumer应平均连接每一个节点，从中取消息。该模式存在一个问题就是当A节点故障后，B节点无法取到A节点中还未消费的消息实体。如果做了队列持久化或消息持久化，那么得等A节点恢复，然后才可被消费，并且在A节点恢复之前其它节点不能再创建A节点已经创建过的持久队列；如果没有持久化的话，消息就会失丢。这种模式更适合非持久化队列，只有该队列是非持久的，客户端才能重新连接到集群里的其他节点，并重新创建队列。假如该队列是持久化的，那么唯一办法是将故障节点恢复起来。
为什么RabbitMQ不将队列复制到集群里每个节点呢？这与它的集群的设计本意相冲突，集群的设计目的就是增加更多节点时，能线性的增加性能（CPU、内存）和容量（内存、磁盘）。理由如下

1、存储空间:如果每个集群节点每个队列的一个完整副本,增加节点需要更多的存储容量。例如,如果一个节点可以存储1 gb的消息,添加两个节点需要两份相同的1 gb的消息
2、性能:发布消息需要将这些信息复制到每个集群节点。对持久消息,要求为每条消息触发磁盘活动在所有节点上。每次添加一个节点都会带来 网络和磁盘的负载。

当然RabbitMQ新版本集群也支持队列复制（有个选项可以配置）。比如在有五个节点的集群里，可以指定某个队列的内容在2个节点上进行存储，从而在性能与高可用性之间取得一个平衡（即镜像模式）。

环境：

|ip|主机名|描述|
|::|::|::|
|192.168.64.128|rabbitmq01|节点1|
|192.168.64.129|rabbitmq02|节点2|
|192.168.64.130|rabbitmq02|节点3


**节点1：**
使用 docker compose 添加rabbitmq 内容如下：
vim docker-compose.yml
```
services:
  rabbitmq:
    image:  rabbitmq:3.7.8-management
    container_name: rabbitmq
    hostname: rabbitmq01
    extra_hosts:
      - "rabbitmq01:192.168.64.128"
      - "rabbitmq02:192.168.64.129"
      - "rabbitmq03:192.168.64.130"
    ports:
      - "5672:5672"
      - "4369:4369"
      - "1883:1883"
      - "15672:15672"
      - "25672:25672"
   volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/data/docker/rabbitmq/data:/var/lib/rabbitmq"
    restart: "always"
```

启动 rabbitmq:
```
[huangkai@sjq128 docker]$ docker-compose up -d rabbitmq
```
启动后，使用浏览器访问 http://192.168.64.128:15672 ，默认账号为: guest ,密码为 guest


**节点2：**
使用 docker compose 添加rabbitmq 内容如下：
vim docker-compose.yml
```
services:
  rabbitmq:
    image: rabbitmq:3.7.8-management
    hostname: rabbitmq02
    extra_hosts:
      - "rabbitmq01:192.168.64.128"
      - "rabbitmq02:192.168.64.129"
      - "rabbitmq03:192.168.64.130"
    ports:
      - "5672:5672"
      - "4369:4369"
      - "1883:1883"
      - "15672:15672"
      - "25672:25672"
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/data/docker/rabbitmq/data:/var/lib/rabbitmq"
    restart: "always"
```
启动rabbitmq服务 `docker-compose up -d rabbitmq`

修改  `/data/docker/rabbitmq/data/.erlang.cookie` 文件内容(在 docker容器中的目录为 `/var/lib/rabbitmq/.erlang.cookie`，这里是因为使用docker 配置了 volumnes )，和 节点1的内容一样,再重启 rabbitmq

```
########在节点1上执行：查看cookie文件内容
[huangkai@sjq128 docker]$ sudo cat rabbitmq/data/.erlang.cookie 
SBIYQJDWNCGPVROAPVOF
[huangkai@sjq128 docker]$ 

############ 在节点2上执行，
[huangkai@sjq129 docker]$ cd rabbitmq/data/
[huangkai@sjq129 data]$ sudo chmod 600 .erlang.cookie # 修改文件权限，默认为只读
[huangkai@sjq129 data]$ echo SBIYQJDWNCGPVROAPVOF > .erlang.cookie #将节点1的cookie信息写入到节点2文件中
[huangkai@sjq129 data]$ sudo chmod 400 .erlang.cookie # 还原文件权限
[huangkai@sjq129 data]$ cd ../.. 
[huangkai@sjq129 docker]$ docker-compose restart rabbitmq
```
进入 rabbitmq 容器: `docker-compose exec rabbitmq bash`

将节点2加入到节点1的集群中: 

```
root@rabbitmq02:/# rabbitmqctl stop_app
Stopping rabbit application on node rabbit@rabbitmq02 ...
root@rabbitmq02:/# rabbitmqctl reset
root@rabbitmq02:/# rabbitmqctl join_cluster --ram rabbit@rabbitmq01  # --ram 表示此节点为内存节点，默认为磁盘节点
Clustering node rabbit@rabbitmq03 with rabbit@rabbitmq01
root@rabbitmq02:/# rabbitmqctl start_app
Starting node rabbit@rabbitmq03 ...
 completed with 3 plugins.
root@rabbitmq02:/# 
```
使用浏览器查看，如下，两个节点都在集群中
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/rabbitmq/rabbitmq_03.png)

查看集群状态信息:

```
root@rabbitmq03:/# rabbitmqctl cluster_status
Cluster status of node rabbit@rabbitmq02 ...
[{nodes,[{disc,[rabbit@rabbitmq01]},
         {ram,[rabbit@rabbitmq02]}]},
 {running_nodes,[rabbit@rabbitmq02,rabbit@rabbitmq01]},
 {cluster_name,<<"rabbit@rabbitmq01">>},
 {partitions,[]},
 {alarms,[{rabbit@rabbitmq01,[]},
          {rabbit@rabbitmq02,[]}]}]
root@rabbitmq03:/#
```

节点3：

节点3 docker-compose 配置内容如下：
```
services:
  rabbitmq:
    image: rabbitmq:3.7.8-management
    hostname: rabbitmq03
    extra_hosts:
      - "rabbitmq01:192.168.64.128"
      - "rabbitmq02:192.168.64.129"
      - "rabbitmq03:192.168.64.130"
    ports:
      - "5672:5672"
      - "4369:4369"
      - "1883:1883"
      - "15672:15672"
      - "25672:25672"
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/data/docker/rabbitmq/data:/var/lib/rabbitmq"
    restart: "always"
```
操作步骤参考节点2.

使用浏览器查看任意节点的 15672 端口如下，三个节点都在集群中

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/rabbitmq/rabbitmq_04.png)


# 二、镜像模式 #