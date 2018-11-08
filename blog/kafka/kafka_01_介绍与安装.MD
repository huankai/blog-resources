---
title: Kafka_01介绍与安装
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

# 一、Kafka 介绍 #

## 1.1介绍 ##

- Apache Kafka是一个开源消息系统，由Scala写成。是由Apache软件基金会开发的一个开源消息系统项目。
- Kafka最初是由LinkedIn公司开发，并于	2011年初开源。2012年10月从Apache Incubator毕业。该项目的目标是为处理实时数据提供一个统一、高通量、低等待的平台。
- Kafka是一个分布式消息队列。Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，此外kafka集群有多个kafka实例组成，每个实例(server)成为broker。
- 无论是kafka集群，还是producer和consumer都依赖于zookeeper集群保存一些meta信息，来保证系统可用性。

## 1.2、 kafka 架构 ##

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_02.png)

- Producer: 消息生产者
- Broker: 一台Kafka服务器就是一个broker，一个集群由多个broker组成，每个broker可以有多个Topic
- Topic :可以理解为一个队列。
- Partition: 为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序，partition可以分为 Leader 与 follower，follower不会与生产者与消费者打交到，只是数据的备份，当这个partition的 leader 无法提供服务时，会从followers 中产 生新的Leader（如上图Topic A 有三个partition,Partiton 1 在Broker 1 上有Leader,在Borkder 2 上有follower）。 
- Offset：kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka
- Consumer ：消息消费者，向kafka broker取消息的客户端
- Consumer Group （CG）：这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制-给consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic。同一个消费组中每个消费者会消费自己的 Topic 中的 Partition，不会存在同一个消费组中多个Consumer 消费中同一个Topic中同一条消息。


## 1.3、分布式模型 ##

1、Kafka每个主题的多个分区日志分布式地存储在Kafka集群上，同时为了故障容错，每个分区都会以副本的方式复制到多个消息代理节点上。其中一个节点会作为主副本（Leader），其他节点作为备份副本（Follower，也叫作从副本）。主副本会负责所有的客户端读写操作，备份副本仅仅从主副本同步数据。当主副本出现故障时，备份副本中的一个副本会被选择为新的主副本。因为每个分区的副本中只有主副本接受读写，所以每个服务器端都会作为某些分区的主副本，以及另外一些分区的备份副本，这样Kafka集群的所有服务端整体上对客户端是负载均衡的。

2、Kafka的生产者和消费者相对于服务器端而言都是客户端。

3、Kafka生产者客户端发布消息到服务端的指定主题，会指定消息所属的分区。生产者发布消息时根据消息是否有键，采用不同的分区策略。消息没有键时，通过轮询方式进行客户端负载均衡；消息有键时，根据分区语义（例如hash）确保相同键的消息总是发送到同一分区。

4，Kafka的消费者通过订阅主题来消费消息，并且每个消费者都会设置一个消费组名称。因为生产者发布到主题的每一条消息都只会发送给消费者组的一个消费者。所以，如果要实现传统消息系统的“队列”模型，可以让每个消费者都拥有相同的消费组名称，这样消息就会负责均衡到所有的消费者；如果要实现“发布-订阅”模型，则每个消费者的消费者组名称都不相同，这样每条消息就会广播给所有的消费者。

5、分区是消费者现场模型的最小并行单位。如下图（图1）所示，生产者发布消息到一台服务器的3个分区时，只有一个消费者消费所有的3个分区。在下图（图2）中，3个分区分布在3台服务器上，同时有3个消费者分别消费不同的分区。假设每个服务器的吞吐量时300MB，在下图（图1）中分摊到每个分区只有100MB，而在下图（图2）中，集群整体的吞吐量有900MB。可以看到，增加服务器节点会提升集群的性能，增加消费者数量会提升处理性能。

6、同一个消费组下多个消费者互相协调消费工作，Kafka会将所有的分区平均地分配给所有的消费者实例，这样每个消费者都可以分配到数量均等的分区。Kafka的消费组管理协议会动态地维护消费组的成员列表，当一个新消费者加入消费者组，或者有消费者离开消费组，都会触发再平衡操作。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_03.png)

# 二、下载与安装： #

## 2.1、 下载： ##

官网下载地址： http://kafka.apache.org/downloads

使用清华大学镜像地址下载：

[huangkai@sjq-10 install]$ **wget http://mirror.bit.edu.cn/apache/kafka/2.0.0/kafka_2.12-2.0.0.tgz**

也可以使用百度云盘下载：
链接：https://pan.baidu.com/s/1GlFnXiWYP0Q9tc059DvqAA 密码：t27x

## 2.2、 解压： ##
 [huangkai@sjq-10 local]$ **sudo tar -xvf kafka_2.12-2.0.0.tgz -C /usr/local/**

## 2.3、配置： ##

- server.properties配置：

**sudo vim /usr/local/kafka_2.12-2.0.0/config/server.properties**

```
# brokerId，在集群中，此brokerId必须是唯一的
broker.id=0
	
# listeners
listeners=PLAINTEXT://sjq-01:9092

# 处理网络请求的线程数量
num.network.threads=3

# 用来处理磁盘IO的线程数量
num.io.threads=8

# 发送套接字的缓冲区大小
socket.send.buffer.bytes=102400

# 接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400

# 请求套接字的最大缓冲区大小
socket.request.max.bytes=104857600

#数据存放目录，可以指定多个目录，用逗号分开，当新的partition创建时，会存放在最少 partition的目录中。
log.dirs=/usr/local/kafka_2.12-2.0.0/logs

# topic在当前broker上的默认分区个数
num.partitions=1

# 用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1

offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

# segment文件保留的最长时间，超时将被删除
log.retention.hours=168

# segment文件大小，超出此数将创建新的文件
log.segment.bytes=1073741824

log.retention.check.interval.ms=300000

#zookeeper连接配置
zookeeper.connect=sjq-01:2181,sjq-01:2181,sjq-01:2181

# zookeeper连接超时配置
zookeeper.connection.timeout.ms=6000

#
group.initial.rebalance.delay.ms=0

是否可以删除topic,在 1.0.0版本中，默认值改为了true
delete.topic.enable=true
```

- producer.properties 配置

**sudo vim /usr/local/kafka_2.12-2.0.0/config/producer.properties**

```
#启动时producer查询brokers的列表，可以是集群中所有brokers的一个子集
bootstrap.servers=localhost:9092

#Partition类，默认对key进行hash，默认为 org.apache.kafka.clients.producer.internals.DefaultPartitioner
partitioner.class=

# Broker等待ack的超时时间，若等待时间超过此值，会返回客户端错误信息
request.timeout.ms=

# 请求延时
linger.ms=

# 一批消息处理大小
batch.size=

# 发送缓存区内存大小
buffer.memory=
```

- consumer.properties 配置

**sudo vim /usr/local/kafka_2.12-2.0.0/config/consumer.properties**

```
#启动时producer查询brokers的列表，可以是集群中所有brokers的一个子集
bootstrap.servers=localhost:9092

#consumer 组id
group.id=test-consumer-group

#
auto.offset.reset=
```

### 2.4、添加环境变量： ###

[huangkai@sjq-10 ~]$ **sudo vim + /etc/profile**
添加如下内容：
```
# KAFKA_HOME
export KAFKA_HOME=/usr/local/kafka_2.12-2.0.0
export PATH=:$KAFKA_HOME/bin:$PATH	
```
使配置文件生效：
[huangkai@sjq-10 ~]$ **source /etc/profile**

### 2.5、启动： ###

在启动 kafka 之前，需要先启动 zookeeper，关于 zookeeper的安装与启动，请点击 这里 查看。 
以守护进程方式：
 [huangkai@sjq-10 ~]$ **kafka-server-start.sh -daemon /usr/local/kafka_2.12-2.0.0/config/server.properties **

 ### 2.6、停止服务： ###

 [huangkai@sjq-10 ~]$ **kafka-server-stop.sh **


 ### 2.7、集群配置： ###
kafka 的集群配置非常简单，只需要将上面 1.3 中配置中的 broker.id 改为集群中全局唯一，  listeners 改为你安装的ip即可，假如上面配置的是 192.168.64.128，我需要在 192.168.64.129 上再配置一台加入 Kafka的集群中，129服务器的配置如下：

```
# brokerId，在集群中，此brokerId必须是唯一的
broker.id=1
	
# listeners
listeners=PLAINTEXT://192.168.64.129:9092

#运行日志目录
log.dirs=/usr/local/kafka_2.12-2.0.0/logs
```

其它配置几乎是相同的，除非你需要个性化配置

 ### 2.8、开机自动启动配置： ###
vim /usr/lib/systemd/system/kafka.service
添加内容如下：
```
[Unit]
Description=kafka
After=network.target
[Service]
Type=forking
User=huangkai
Group=huangkai
ExecStart=/usr/local/kafka_2.12-2.0.0/bin/kafka-server-start.sh -daemon /usr/local/kafka_2.12-2.0.0/config/server.properties
ExecReload=/usr/local/kafka_2.12-2.0.0/bin/kafka-server-restart.sh
ExecStop=/usr/local/kafka_2.12-2.0.0/bin/kafka-server-stop.sh
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
 ### 2.9、kafka重启脚本： ###

[huangkai@sjq-01 ~]$ **vim /usr/local/kafka_2.12-2.0.0/bin/kafka-server-restart.sh**
```
#!/bin/bash

base_dir=$(dirname $0)

exec $base_dir/kafka-server-stop.sh

sleep 2

exec $base_dir/kafka-server-start.sh -daemon $base_dir/../config/server.properties
```


# 三、Kafka工作流程分析： #

## 3.1、写入方式 ##

生产者(producer)采用推（push）模式将消息发布到broker，每条消息都被追加（append）到分区（patition）中，属于顺序写磁盘（顺序写磁盘效率比随机写内存要高，保障kafka吞吐率）。

## 3.2、分区(partition) ##

1、Kafka集群有多个消息代理服务器（broker-server）组成，发布到Kafka集群的每条消息都有一个类别，用主题（topic）来表示。通常，不同应用产生不同类型的数据，可以设置不同的主题。一个主题一般会有多个消息的订阅者，当生产者发布消息到某个主题时，订阅了这个主题的消费者都可以接收到生成者写入的新消息。

2、Kafka集群为每个主题维护了分布式的分区（partition）日志文件，物理意义上可以把主题（topic）看作进行了分区的日志文件（partition log）。主题的每个分区都是一个有序的、不可变的记录序列，新的消息会不断追加到日志中。分区中的每条消息都会按照时间顺序分配到一个单调递增的顺序编号，叫做偏移量（offset），这个偏移量能够唯一地定位当前分区中的每一条消息。

3、消息发送时都被发送到一个topic，其本质就是一个目录，而topic是由一些Partition Logs(分区日志)组成，下图中的topic有3个分区，每个分区的偏移量都从0开始，不同分区之间的偏移量都是独立的，不会相互影响。


我们可以看到，每个Partition中的消息都是有序的，生产的消息被不断追加到Partition log上，其中的每一个消息都被赋予了一个唯一的offset值。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_04.png)

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_05.png)

4、发布到Kafka主题的每条消息包括键值和时间戳。消息到达服务器端的指定分区后，都会分配到一个自增的偏移量。原始的消息内容和分配的偏移量以及其他一些元数据信息最后都会存储到分区日志文件中。消息的键也可以不用设置，这种情况下消息会均衡地分布到不同的分区。

### 3.2.1、分区原因 ###

- 方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；
- 可以提高并发，因为可以以Partition为单位读写。
  传统消息系统在服务端保持消息的顺序，如果有多个消费者消费同一个消息队列，服务端会以消费存储的顺序依次发送给消费者。但由于消息是异步发送给消费者的，消息到达消费者的顺序可能是无序的，这就意味着在并行消费时，传统消息系统无法很好地保证消息被顺序处理。虽然我们可以设置一个专用的消费者只消费一个队列，以此来解决消息顺序的问题，但是这就使得消费处理无法真正执行。

  Kafka比传统消息系统有更强的顺序性保证，它使用主题的分区作为消息处理的并行单元。Kafka以分区作为最小的粒度，将每个分区分配给消费者组中不同的而且是唯一的消费者，并确保一个分区只属于一个消费者，即这个消费者就是这个分区的唯一读取线程。那么，只要分区的消息是有序的，消费者处理的消息顺序就有保证。每个主题有多个分区，不同的消费者处理不同的分区，所以Kafka不仅保证了消息的有序性，也做到了消费者的负载均衡

### 3.2.2、分区原则 ###
	
- 如果指定了patition，则直接使用；
- 如果未指定patition但指定key，通过对key的value进行hash出一个patition
- 如果patition和key都未指定，使用轮询选出一个patition，可以查看kafka API (DefaultPartitioner)

### 3.2.3、分区副本 ###

同一个分区(partition)可能会有多个replication（对应 server.properties 配置中的 default.replication.factor=N）。没有replication的情况下，一旦broker 宕机，其上所有 patition 的数据都不可被消费，同时producer也不能再将数据存于其上的patition。引入replication之后，同一个partition可能会有多个replication，而这时需要在这些replication之间选出一个leader，producer和consumer只与这个leader交互，其它replication作为follower从leader 中复制数据。

## 3.3、Broker 保存消息 ##

### 3.3.1、存储方式 ###
物理上把topic分成一个或多个patition（对应 server.properties 中的num.partitions=3配置），每个patition物理上对应一个文件夹（该文件夹存储该patition的所有消息和索引文件），可以查看 ${KAFKA_HOME}/logs 目录。

### 3.3.2、存储策略 ###
无论消息是否被消费，kafka都会保留所有消息。有两种策略可以删除旧数据(见 ${KAFKA_HOME}/config/server.properties配置，默认是基于时间)：

- 基于时间：log.retention.hours=168
- 基于大小：log.retention.bytes=1073741824

需要注意的是，因为Kafka读取特定消息的时间复杂度为O(1)，即与文件大小无关，所以这里删除过期文件与提高 Kafka 性能无关。


### 3.3.3、zookeeper存储结构 ###

zookeeper 不会存储kafka 生产者信息，borker 信息和消费者信息会在zookeeper中注册，可以使用 zooInspector  工具连接查看，结构如下图所示：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_06.png)


上图中的绿色节点，是可以在开发中根据需求编辑的，如topic的修改、重置消费者偏移量等。

## 3.4、消费模型 ##

### 3.4.1、消费模型 ###
消息由生产者发布到Kafka集群后，会被消费者消费。消息的消费模型有两种：推送模型（push）和拉取模型（pull）。

基于推送模型（push）的消息系统，由消息代理记录消费者的消费状态。消息代理在将消息推送到消费者后，标记这条消息为已消费，但这种方式无法很好地保证消息被处理。比如，消息代理把消息发送出去后，当消费进程挂掉或者由于网络原因没有收到这条消息时，就有可能造成消息丢失（因为消息代理已经把这条消息标记为已消费了，但实际上这条消息并没有被实际处理）。如果要保证消息被处理，消息代理发送完消息后，要设置状态为“已发送”，只有收到消费者的确认请求后才更新为“已消费”，这就需要消息代理中记录所有的消费状态，这种做法显然是不可取的。

Kafka采用拉取模型，由消费者自己记录消费状态，每个消费者互相独立地顺序读取每个分区的消息。如下图所示，有两个消费者（不同消费者组）拉取同一个主题的消息，消费者A的消费进度是3，消费者B的消费进度是6。消费者拉取的最大上限通过最高水位（watermark）控制，生产者最新写入的消息如果还没有达到备份数量，对消费者是不可见的。这种由消费者控制偏移量的优点是：消费者可以按照任意的顺序消费消息。比如，消费者可以重置到旧的偏移量，重新处理之前已经消费过的消息；或者直接跳到最近的位置，从当前的时刻开始消费。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_07.png)

在一些消息系统中，消息代理会在消息被消费之后立即删除消息。如果有不同类型的消费者订阅同一个主题，消息代理可能需要冗余地存储同一消息；或者等所有消费者都消费完才删除，这就需要消息代理跟踪每个消费者的消费状态，这种设计很大程度上限制了消息系统的整体吞吐量和处理延迟。Kafka的做法是生产者发布的所有消息会一致保存在Kafka集群中，不管消息有没有被消费。用户可以通过设置保留时间来清理过期的数据，比如，设置保留策略为两天。那么，在消息发布之后，它可以被不同的消费者消费，在两天之后，过期的消息就会自动清理掉。


### 3.4.2、高级API ###

高级API优点：

- 高级API 写起来简单； 
- 不需要自行去管理offset，系统通过zookeeper自行管理。
- 不需要管理分区，副本等情况，.系统自动管理。
- 消费者断线会自动根据上一次记录在zookeeper中的offset去接着获取数据（默认设置1分钟更新一下zookeeper中存的offset）
- 可以使用group来区分对同一个topic 的不同程序访问分离开来（不同的group记录不同的offset，这样不同程序读取同一个topic才不会因为offset互相影响）

高级API缺点：

- 对于某些特殊需求来说，不能自行控制offset； 
- 不能细化控制如分区、副本、zk等


### 3.4.3、低级API ###

低级 API 优点：
- 能够让开发者自己控制offset，想从哪里读取就从哪里读取。
- 自行控制连接分区，对分区自定义进行负载均衡
- 对zookeeper的依赖性降低（如：offset不一定非要靠zk存储，自行存储offset即可，比如存在文件或者内存中）

低级API缺点：

- 太过复杂，需要自行控制offset，连接哪个分区，找到分区leader 等

### 3.4.2、消费方式 ###

<font color='red'>消费者consumer采用pull（拉）模式从broker中读取数据。 </font>

1、push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。它的目标是尽可能以最快速度传递消息，但是这样很容易造成consumer来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而pull模式则可以根据consumer的消费能力以适当的速率消费消息。

2、对于Kafka而言，pull模式更合适，它可简化broker的设计，consumer可自主控制消费消息的速率，同时consumer可以自己控制消费方式——即可批量消费也可逐条消费，同时还能选择不同的提交方式从而实现不同的传输语义。

3、pull模式不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直等待数据到达。为了避免这种情况，我们在拉请求中有参数，允许消费者请求在等待数据到达的“长轮询”中进行阻塞（并且可选地等待到给定的字节数，以确保大的传输大小）。