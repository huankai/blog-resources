---
title: Redis 发布与订阅
date: {{ date }}
author: huangkai
tags: 
	- Redis
---

Redis发布与订阅

订阅，取消订阅和发布实现了发布/订阅消息范式，发送者（发布者）不是计划发送消息给特定的接收者（订阅者）。而是发布的消息分到不同的频道，不需要知道什么样的订阅者订阅。订阅者对一个或多个频道感兴趣，只需接收感兴趣的消息，不需要知道什么样的发布者发布的。这种发布者和订阅者的解耦合可以带来更大的扩展性和更加动态的网络拓扑。

Redis的发布与订阅功能说白了就是消息，它支持消息功能 ，但是在企业开发中一般不会选择使用Redis的发布与订阅，而会选择消息中间件，如 [ActiveMq](http://activemq.apache.org/)、[RabbitMq](http://www.rabbitmq.com/) 、[RocketMq](http://rocketmq.apache.org/)　、[Kafka](http://kafka.apache.org/)等。在企业开发中Redis的主要还是作为分布式内存缓存使用。

发布与订阅命令：


|命令|语法|开始版本|描述|
|:---:|:---:|:---:|---|
|PSUBSCRIBE|PSUBSCRIBE pattern [pattern ...]|2.0.0|订阅给定的模式(patterns)|
|PUBSUB|PUBSUB subcommand [argument [argument ...]]|2.8.0|PUBSUB是一个内省命令，允许检查Pub / Sub子系统的状态。它由单独记录的子命令组成| 
|PUBLISH|PUBLISH channel message|2.0.0|将信息 message 发送到指定的频道 channe|
|PUNSUBSCRIBE|PUNSUBSCRIBE [pattern [pattern ...]]|2.0.0|停止发布到匹配给定模式的渠道的消息监听|
|SUBSCRIBE|SUBSCRIBE channel [channel ...]|2.0.0|订阅给指定频道的信息|
|UNSUBSCRIBE|UNSUBSCRIBE [channel [channel ...]]|2.0.0|停止频道监听|


# 1、SUBSCRIBE(订阅频道) #

一个客户端订阅 频道 channel1 与 channel2

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Redis/pubsub/1.png)

# 2、PUBLISH(发布消息) #

在 channel1 发布消息 hello

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Redis/pubsub/2.png)

此时订阅了 channel1频道的客户端将收到发送的消息，见 第一张图片

# 3、PSUBSCRIBE(通配符订阅模式) #


通配符订阅以 new 开头的频道

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Redis/pubsub/3.png)

发送消息到 new1 和 new13 频道，上面可以收到信息

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Redis/pubsub/4.png)







