---
title: Spring Cloud Eureka 服务发现
date: {{ date }}
author: huangkai
tags:
    - Spring Cloud
---

Spring Cloud是一套基于Spring Boot 实现的微服务开发工具，微服务(也称微服务架构)，简单地说，就是将一个系统按照一定的规则有效的拆分成多个不同的服务，每个服务都能独立的进行开发、管理、扩展、维护。服务与服务之间可以通过Restful API 等方式进行相互调用。

Spring Cloud并没有重复制造轮子，它只是将业界多个开源的微服务框架集成起来，再通过Spring Boot进行包装屏蔽复杂的配置和实现原理，目的就是给开发者一套简单易懂、易部署与维护的分布式系统开发工具包。它提供了微服务开发所需要的配置管理、服务发现、负载均衡、断路器、智能路由、微代理、控制总线等组件。

## 1、Eureka ##

Eureka是一种基于REST 	的服务，主要用于定位服务，以实现中间层服务器的负载均衡和故障转移。它是由 Spring Cloud Netflix(Spring Cloud 子项目)项目提供的。

### 1.1、Spring Cloud Netflix ###

它主要是对 Netflix 开源的一系列产品进行包装，为 Spring Boot 应用程序提供自动配置的 Netflix OSS 集成。通过一些简单的注解，就能快速启用并构建大型的分布式系统。它提供的模块有：
服务发现（Eureka）、断路器（Hystrix）、智能路由（Zuul）、客户端负载均衡（Ribbon）。

### 1.2、样例项目结构 ###
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/SpringCloud/01.png)
### 1.3、服务注册中心 ###
在 pom.xml 中声明使用spring-cloud-starter-eureka-server启动器（本示例对应的项目是eureka-server）
```
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka-server</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-log4j2</artifactId>
	</dependency>
</dependencies>
```
使用@EnableEurekaServer注解将应用声明为 Eureka 服务器端（Eureka Server），从而启动 Eureka 服务注册中心的组件，对外提供服务注册和发现的功能。
```
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServerApplication.class, args);
	}

}
```

在默认的模式中，Eureka 服务器端也充当客户端并向给定的 serviceUrl 注册自己。在生产环境中，我们通常会有多台服务器端应用，但是为了简单起见，本示例使用单台服务器，因此需要禁掉 Eureka 服务器端应用的客户端行为：

```
### 坐标： ${hk-eureka-simple}/src/main/resources/application.yml ###
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
```
|配置项|默认值|说明|
|:---:|:---:|:---:|:---:|
|eureka.instance.hostname|-|实例的主机名|
|eureka.client.register-with-eureka|true|该实例是否向 Eureka 服务器注册自己，以供外部应用发现自己。在某些情况下，你可能不希望当前的应用被外部的其他应用发现，而只是想从服务器发现其他服务的实例，此时你可以将此值设为 false|
|eureka.client.fetch-registry	|true|该实例是否向 Eureka 服务器获取所有的注册信息表|
|eureka.client.service-url.defaultZone|-|该实例与 Eureka 服务器通讯的 URL 地址列表。如果 Eureka 服务器地址不止一个，则使用英文的逗号分隔|

Eureka 服务器默认监听 8761 端口来接收服务注册，除此之外它还提供一个可视化的直观页面，可以方便的查看注册的服务。EurekaServerApplication，访问：http://localhost:8761
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/SpringCloud/02.png)

从上图可以看到，此时还没有任何服务注册到 Eureka 服务器。

### 1.4、客户端（服务提供者） ###

在 pom.xml 中声明使用spring-cloud-starter-eureka启动器（本示例对应的项目是user-service）：

```
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-eureka</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-log4j2</artifactId>
	</dependency>
</dependencies>
```

使用@EnableEurekaClient或@EnableDiscoveryClient注解可以将应用声明为 Eureka 客户端（Eureka Client）。当客户端向 Eureka 服务器注册时，它会提供关于自身的一些元数据，例如主机和端口，健康指示符 URL，主页等信息。

```
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {

	public static void main(String[] args) {
		SpringApplication.run(UserServiceApplication.class, args);
	}

}
```
还需要配置才能找到 Eureka 服务器：
```
### 坐标： ${hk-user-service}/src/main/resources/application.yml ###
server:
  port: 8881

spring:
  application:
    name: user-service

eureka: 
  client:
    service-url: 
      defaultZone: http://localhost:8761/eureka
```

spring.application.name是 Eureka 客户端向服务器注册的服务ID和虚拟主机的名称。在 Eureka 服务器中，服务ID相同的实例将集群在一起。
UserServiceApplication，再次访问：http://localhost:8761

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/SpringCloud/03.png)

从上图可以看到，客户端应用程序已经成功被注册了。

### 1.5、高可用 ###
以上示例都是单点运行的，不适合于生产环境。[Eureka](https://github.com/Netflix/eureka/wiki/Eureka-at-a-glance#high-level-architecture) 官方给出的应用部署架构图是这样的：

![](https://raw.githubusercontent.com/Netflix/eureka/master/images/eureka_architecture.png)

下面对这个架构图来作一些解读，希望能帮助你更好的理解。
** 1.5.1、Region 和 Zone **
在 Eureka 中有 Region（区域）和 Zone（Availability Zone，可用区）的区分。
这是由于 Netflix 开源的 Eureka 旨在 AWS（Amazon Web Services，现在通常称为云计算）中运行，因此使用了一些 AWS 特有的概念术语。
在非 AWS 的环境下，我们可以简单的将 Region 理解成大区或地域（如阿里云服务器的华南、华北地区），Zone 可以简单的理解成机房。如需了解更多相关信息，可参考：[AWS的区域和可用区概念解释](http://blog.csdn.net/awschina/article/details/17639191)
上图是一个 Eureka 集群的部署架构图，它面有 3 个 Zone（us-east-1c、us-east-1d、us-east-1e），它们都属于 us-east-1 这个 Region。

** 1.5.2、Eureka Server **
每个 Zone 至少有一个 Eureka Server，能够对外提供服务发现和处理区域故障。
在 Eureka Server 集群中，没有 Master/Slave 的区分，每个Eureka Server都是对等(Peer)的。它们除了可以作为服务注册中心外，也可以充当客户端向其它 Eureka Server 注册自己，并会从他们对等的节点(由 erueka.client.service-url.defaultZone配置指定)中复制(replicate) 所有服务注册表信息以达到同步的目的，如果因为某种原因导致失败，默认等待 5 分钟(可以通过 erueka.server.wait-time-in-ms-when-sync-empt配置)，在这期间，它不向客户端提供服务注册信息。并且默认失败重试 5 次(可以通过eureka.server.number-of-replication-retries配置)。

** 1.5.3、Eureka Client** 
Eureka 客户端应用分为两种，Application Service(服务提供者) 与 Application Client(服务消费者)。
Application Service 通常需要向给定 serviceUrl 对应的Eureka Server 来注册自己，以供外部应用可以发现自己。其注册信息包括主机名和端口信息等元数据。然后默认以每隔30秒的频率向Erueka Server发送一次心跳(可以通过eureka.instance.lease-renewal-interval-in-seconds 配置)来续约服务。

Eureka Server默认为 90 秒内如果没有收到客户端的心跳，则它会将该客户端实例从它的注册表中剔除，以禁止该实例的流量(可以通过erueka.instance.lease-expiration-duration-in-seconds配置，注意：如果该值设置过大，即使实例不存在了，流量也可以路由到该实例；如果设置过小可能因为网络原因导致实例被服务器剔除；该值至少应该比发送心跳频率的间隔值要大)。

Eureka客户端默认会从注册的 Eureka Server中获取所有的服务注册表信息(可以通过eureka.client.fetch-registry配置)，默认是以每隔30秒获取注册表信息(可以通过eureka.client.registry-fetch-interval-seconds配置)。

Application Client 可以不向任何Eureka Server注册自己，它可以只从Eureka Server获取注册过的服务列表，通过RESTFUL API 的方式远程调用服务提供者。

**1.5.4、 Eureka Server 高可用样例**
本示例在同一主机运行多个Eureka Server 实例，由于Eureka 会过滤同一主机的相同主机名(详见 ``com.netflix.eureka.cluster.PeerEurekaNodes#isThisMyUrl``)，但它不检查端口，因此需要先定义至少两个不同的主机名，并使它们映射到``127.0.0.1``，这里采用修改 hosts文件的方式，Windows系统 hosts文件路径在``C:\Windows\System32\drivers\etc\hosts`` 打开此文件，在最后一行添加：
```
127.0.0.1 peer1 peer2 peer3
```
复制 hk-eureka-simple 为 hk-eureka-highavailability,修改 ${hk-eureka-highavailability}/src/main/resources/application.yml配置如下：
```
spring:
  application:
    name: eureka-server-highavailability
  profiles:
    active:
    - peer1
    
logging:
  config: classpath:log4j2.xml 
  
---
spring:
  profiles: peer1
server:
  port: 8761
  
eureka:
  instance:
    hostname: peer1
  client:
    service-url:
      defaultZone: http://peer2:8762/eureka,http://peer3.8763/eureka
---
spring:
 profiles: peer2
server:
  port: 8762
eureka: 
  instance:
    hostname: peer2
  client:
    service-url:
      defaultZone: http://peer1:8761/eureka,http://peer3.8763/eureka
---
spring:
 profiles: peer3
server:
  port: 8763
eureka: 
  instance:
    hostname: peer3
  client:
    service-url:
      defaultZone: http://peer1:8761/eureka,http://peer2.8762/eureka
```

这里配置了3 个Eureka Server实例 ，每个实例与其它两个实例分别进行两两相互注册关系图如下：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/SpringCloud/04.png)

需要注意的是：Eureka Server的服务注册信息不能二次传播，如下图的实例关系配置是不可取的

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/SpringCloud/05.png)

上图的每个Eureka Server实例是单向向另外一个实例注册，如果现有一个新的客户端实例C 向 Server1 注册， Server1 与 Server2中都会有C 的注册信息，但是Server3 中没有C的注册信息(详见:``com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#replicateToPeers``)。

启动3个Eureka Server实例：
- 使用 Jar方式启动
先将项目用maven打成 jar包，cmd 进入jar所在目录，
```
# 启动 eureka 实例1
java -jar hk-eureka-highavailability-0.0.1-SNAPSHOT.jar
# 启动 eureka 实例2
java -jar -Dspring.profiles.active=peer2 hk-eureka-highavailability-0.0.1-SNAPSHOT.jar
# 启动 eureka 实例3
java -jar -Dspring.profiles.active=peer3 hk-eureka-highavailability-0.0.1-SNAPSHOT.jar
```
- 使用 STS/Eclipse 启动
```
# 启动 eureka 实例1
右键 -> Run As -> Spring Boot App
# 启动 eureka 实例2
右键 -> Run As -> Run Configurations -> 左边 Spring Boot App ->右键 -> NEW -> 右边 Spring Boot 选项卡指定 project 与 Main type ,Arguments 选项卡中的 Program arguments 中添加 --spring.profiles.active=peer2 -> 右下角点击 Run 运行
# 启动 eureka 实例3
右键 -> Run As -> Run Configurations -> 左边 Spring Boot App ->右键 -> NEW -> 右边 Spring Boot 选项卡指定 project 与 Main type ,Arguments 选项卡中的 Program arguments 中添加 --spring.profiles.active=peer3 -> 右下角点击 Run 运行
```
启动 eureka 实例1与实例2时，可能会有连接的错误信息，因为无法连接上实例3，当实例1与实例2都启动成功后，在启动实例3时不会有错误信息存在。

浏览器访问 ``http://localhost:8761``:

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/SpringCloud/06.png)

**1.5.5、 Eureka Client 高可用样例**

复制 hk-user-service 为 hk-user-service-highavailability,修改 ${hk-user-service-highavailability}/src/main/resources/application.yml配置如下：
```
spring:
  application:
    name: user-service-highavailability
  profiles:
    active: client1
eureka:
  client:
    service-url:
      defaultZone: http://peer1:8761/eureka/
---
spring:
  profiles: client1
server:
  port: 8881
---
spring:
  profiles: client2
server:
  port: 8882
```
客户端的eureka.client.service-url.defaultZone指定为当前 Zone 中任意一台服务注册中心的地址就可以，因为上例中配置的每台服务注册中心的服务注册表是两两相互进行复制的。
启动 2 个 Eureka Client 实例:
```
# 启动 client 实例1
右键 -> Run As -> Spring Boot App
# 启动 client 实例2
右键 -> Run As -> Run Configurations -> 左边 Spring Boot App ->右键 -> NEW -> 右边 Spring Boot 选项卡指定 project 与 Main type ,Arguments 选项卡中的 Program arguments 中添加 --spring.profiles.active=client2 -> 右下角点击 Run 运行
```

浏览器重新刷新``http://localhost:8761``:

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/SpringCloud/07.png)

### 1.6、自我保护模式 ###
Eureka 默认开启了自我保护模式（可以通过eureka.server.enable-self-preservation配置）。该模式被激活的条件是：在 1 分钟后，Renews (last min)<Renews threshold。你可以在 Eureka Server 首页的右上侧可以看到：

![](https://raw.githubusercontent.com/fanlychie/mdimg/master/spring-cloud-netflix-eureka-server-page-part.png)

|参数名|描述|
|:---:|:---:|
|Renews threshold|Eureka Server 期望每分钟收到客户端实例续约的总数|
|Renews (last min)|Eureka Server 最后 1 分钟收到客户端实例续约的总数|

**1.6.1、服务器端续约阀值的计算源码（Renews threshold）**


```
# com.netflix.eureka.registry.PeerAwareInstanceRegistryImpl#openForTraffic #
this.expectedNumberOfRenewsPerMin = count * 2;
this.numberOfRenewsPerMinThreshold = (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
```
其中，count 为 服务器的数量。数值 2 表示每 30 秒 1 个心跳，每分钟 2 个心跳的固定频率因子。

归纳公式：2M * renewalPercentThreshold。其中，M 为服务器的个数，计算结果只保留整数位。

renewalPercentThreshold 默认是 0.85（可以通过eureka.server.renewal-percent-threshold配置）。

其实这就是个固定值，因为对于每个 Eureka Server 来说，M 只能取 1。这段代码达到的效果是：

1．expectedNumberOfRenewsPerMin 重置为固定值 2；
2．numberOfRenewsPerMinThreshold 的值被设置为 1；

**1.6.2、客户器端续约阀值的计算源码（Renews threshold）**

```
# com.netflix.eureka.registry.AbstractInstanceRegistry#register #
if (this.expectedNumberOfRenewsPerMin > 0) {
    this.expectedNumberOfRenewsPerMin = this.expectedNumberOfRenewsPerMin + 2;
    this.numberOfRenewsPerMinThreshold = (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
}
```
注：上面贴出的 PeerAwareInstanceRegistryImpl 继承自 AbstractInstanceRegistry。
它们共享 expectedNumberOfRenewsPerMin 和 numberOfRenewsPerMinThreshold 属性，具体可自行翻阅源码。

设有 N 个客户端，服务器端先启动，expectedNumberOfRenewsPerMin 被重置为固定值 2。接着客户端依次启动：
``N = 1–>(2 + 2) * renewalPercentThreshold``
``N = 2–>(2 + 2 + 2) * renewalPercentThreshold``
``N = 3–>(2 + 2 + 2 + 2) * renewalPercentThreshold``
归纳公式：``2(N + 1) * renewalPercentThreshold``，计算结果只保留整数位。
即，如果只有 1 个 Eureka Server 或者有多个 Eureka Server 但它们之间没有相互注册：

当 N = 0 时，只计算服务器端。``Renews threshold= 1``。由于没有客户端向服务器发送心跳，``Renews (last min)<Renews threshold``，Eureka 自我保护模式被激活；
当 N ≠ 0 时，服务器端的计算结果被客户端覆盖，即只计算客户端；
当 N = 2 时，``Renews threshold= 2(N + 1) * renewalPercentThreshold = 2 * 3 * 0.85 = 5``。2 个客户端以每 30 秒发送 1 个心跳，1 分钟后总共向服务器发送 4 个心跳，``Renews (last min)<Renews threshold``，Eureka 自我保护模式被激活；
所以如果 N < 3，在 1 分钟后，服务器端收到的客户端实例续约的总数总是小于期望的阀值，因此 Eureka 的自我保护模式自动被激活。首页会出现警告信息：
<font color='red'>EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY’RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.</font>

这种情况下，由于 Eureka Server 没有对等的节点，同步不到服务注册信息，默认需等待 5 分钟（可以通过eureka.server.wait-time-in-ms-when-sync-empty配置）。即 5 分钟后你应该看到此信息。

为避免这种情况发生，你可以：
- 关闭自我保护模式（eureka.server.enable-self-preservation设为 false）
- 降低 renewalPercentThreshold 的比例（eureka.server.renewal-percent-threshold设置为 0.5 以下，比如 0.49）
- 部署多个 Eureka Server 并开启其客户端行为（eureka.client.register-with-eureka不要设为 false，默认为 true）

如果是采取部署多个 Eureka Server 并开启其客户端行为使其相互注册。假设有 M 个 Eureka Server，那么，每个 Eureka Server 每分钟可以额外收到 2 * (M - 1) 个心跳。例如：
当 M = 1，N = 2 时，``Renews threshold= 2(N + 1) * renewalPercentThreshold = 2 * 3 * 0.85 = 5``，2 个客户端以每 30 秒发送 1 个心跳，1 分钟后总共向服务器发送 4 个心跳，``Renews (last min)<Renews threshold``；
当 M = 2，N = 2 时，``Renews threshold= 2(N + 1) * renewalPercentThreshold = 2 * 3 * 0.85 = 5``，2 个客户端以每 30 秒发送 1 个心跳，1 分钟后总共向服务器发送 4 个心跳，另外还有 1 个 M 发来的 2 个心跳，总共是 6 个心跳，``Renews (last min)>Renews threshold``；

Eureka 的自我保护模式是有意义的，该模式被激活后，它不会从注册列表中剔除因长时间没收到心跳导致租期过期的服务，而是等待修复，直到心跳恢复正常之后，它自动退出自我保护模式。这种模式旨在避免因网络分区故障导致服务不可用的问题。例如，两个客户端实例 C1 和 C2 的连通性是良好的，但是由于网络故障，C2 未能及时向 Eureka 发送心跳续约，这时候 Eureka 不能简单的将 C2 从注册表中剔除。因为如果剔除了，C1 就无法从 Eureka 服务器中获取 C2 注册的服务，但是这时候 C2 服务是可用的。所以，Eureka 的自我保护模式最好还是开启它。

### 1.7、Eureka 与 Zookeeper 的区别 ###
Eureka 最大程度上保证 AP（Availability，可用性；Partition-tolerance，分区容错性），而 Zookeeper 保证的是 CP（Consistency，一致性；Partition-tolerance，分区容错性）。
如果因为网络分区故障导致服务器（master 节点）无法与其它节点联系，对于 Zookeeper 来说，这是不能容忍的。它会对剩下的节点重新进行 leader 选举，在这期间，整个 Zookeeper 集群是不可用的，这就直接导致了所有注册服务瘫痪的现象。
而对于 Eureka 来说，每个节点都是对等的，失去了一个节点，就自动切换到其它节点，只要还有一个 Eureka 节点存在，就能正常对外提供注册服务。Eureka 可以很好的应对因网络分区故障而导致的部分节点失去联系的状况。