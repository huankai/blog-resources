---
title: Redis 集群
date: {{ date }}
author: huangkai
tags: 
	- Redis
---

# 一、安装环境: #

操作系统: Centos Linux 7.3.1611 (Core)
Redis版本：4.0.1
Redis集群配置:

|主机名|Ip地址|端口号|集群端口号(<font color='red'>默认为端口号 + 10000)</font>|
|:-:|:-:|:-:|:-:|
|sjq-01|192.168.64.128(master)|6379|16379|
|sjq-01|192.168.64.128(master)|6380|16380|
|sjq-01|192.168.64.128(slave)|6381|16381|
|sjq-02|192.168.64.129(master)|6379|16379|
|sjq-02|192.168.64.129(slave)|6380|16380|
|sjq-02|192.168.64.129(slave)|6381|16381|

其中192.168.64.128:6379(master) 的 slave 为 192.168.64.129:6380 (slave)
192.168.64.129:6379 (master) 的 slave 为 192.168.64.128:6381 (slave)
192.168.64.128:6380 (master) 的 slave 为 192.168.64.129:6381 (slave)


# 二、安装Redis # 
请 [点击这里](https://huankai.github.io/2018/02/22/Redis_01_%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B/) 查看Redis的安装

# 三、创建集群 #
## 3.1、创建集群 ##
每台服务器安装 ``ruby`` 和 ``rubyge	ms``
使用 yum 安装 ： <font color='red'>``yum -y install ruby rubygems``</font>
查看ruby 版本： <font color='red'>`ruby -v`</font>

gem 安装 redis ruby 接口 ：  <font color='red'>``gem install redis``</font>

01.png

如果出现以上错误，表示redis 4.0.1需要的ruby版本至少大于等2.2.2，而上面通过yum安装的ruby版本为2.0 。
解决办法如下：
- 1、安装 curl :<font color='red'>``yum -y install curl``</font>
- 2、安装RVM:
先执行：<font color='red'>``gpg2 --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3``</font>
再执行：<font color='red'>``curl -L get.rvm.io | bash -s stable``</font>
02.png
在安装rvm时，可能会有连接不到远程服务器的问题，这要就要看你的人品了，有时候不会出现这样的问题，目录没有找到解决办法，等一会再执行，好像就可以了。
- 3、加载配置文件：<font color='red'>``source /usr/local/rvm/scripts/rvm``</font>
- 4、查看rvm库中已知的ruby版本：<font color='red'>``rvm list known``</font>
- 5、安装一个ruby版本： <font color='red'>``rvm install 2.3.3``</font>  ，这里我安装的为2.3.3，只要大于2.2.2就可以
- 6、使用一个ruby版本：<font color='red'>``rvm use 2.3.3``</font>
- 7、卸载一个已知版本：<font color='red'>``rvm remove 2.0.0``</font>
- 8、查看ruby 版本: <font color='red'>``ruby -v``</font> ，此时 ruby 版本成了 2.3.3
- 9、再使用 <font color='red'>``gem install redis``</font> 安装ruby redis接口，就不会有上面的错误信息了

配置Redis集群节点信息（每台服务器执行）：
```
[root@sjq-02 redis-4.0.1]# mkdir -p conf/cluster/6379
[root@sjq-02 redis-4.0.1]# mkdir -p conf/cluster/6380
[root@sjq-02 redis-4.0.1]# mkdir -p conf/cluster/6381
[root@sjq-02 redis-4.0.1]# cp redis.conf conf/cluster/6379/
[root@sjq-02 redis-4.0.1]# cp redis.conf conf/cluster/6380/
[root@sjq-02 redis-4.0.1]# cp redis.conf conf/cluster/6381/
[root@sjq-02 redis-4.0.1]# cd conf/cluster/6379/
[root@sjq-02 6379]# ll
total 60
-rw-r--r--. 1 root root 57764 Mar 14 11:14 redis.conf
[root@sjq-02 6379]#
```

使用vim修改 6379目录 下的redis.conf配置信息如下：

|**参数名称**|**参数值**|**描述**|
|:--:|:--:|:--:|
|bind|192.168.64.129|Bind ip|
|protected-mode|yes|开启保护模式|
|port|6379|redis端口号|
|daemonize|yes|守护进程启动|
|pidfile|/var/run/redis_6379.pid|pid文件|
|dir|/usr/local/redis-4.0.1/conf/cluster/6379|数据存放目录|
|databases|1|默认为16，集群时设置一个即可|
|cluster-enabled|yes|开启集群|
|cluster-config-file|/usr/local/redis-4.0.1/config/cluster/6379/nodes.conf|集群配置文件（启动时自动生成在当前目录下，建议在6379目录下启动redis节点）|
|cluster-node-timeout|15000|节点互联超时时间，毫秒|
|cluster-migration-barrier|1|数据迁移的副本临界数，这个参数表示一个主节点在拥有多少个从节点时就要割让一个从节点给另一个没有任何从节点的主节点|

使用vim修改 6380目录 下的redis.conf配置信息如下：

|**参数名称**|**参数值**|**描述**|
|:--:|:--:|:--:|
|bind|192.168.64.129|Bind ip|
|protected-mode|yes|开启保护模式|
|port|6380|redis端口号|
|daemonize|yes|守护进程启动|
|pidfile|/var/run/redis_6380.pid|pid文件|
|dir|/usr/local/redis-4.0.1/conf/cluster/6380|数据存放目录|
|databases|1|默认为16，集群时设置一个即可|
|cluster-enabled|yes|开启集群|
|cluster-config-file|/usr/local/redis-4.0.1/config/cluster/6380/nodes.conf|集群配置文件（启动时自动生成在当前目录下，建议在6379目录下启动redis节点）|
|cluster-node-timeout|15000|节点互联超时时间，毫秒|
|cluster-migration-barrier|1|数据迁移的副本临界数，这个参数表示一个主节点在拥有多少个从节点时就要割让一个从节点给另一个没有任何从节点的主节点|

使用vim修改 6381目录 下的redis.conf配置信息如下：

|**参数名称**|**参数值**|**描述**|
|:--:|:--:|:--:|
|bind|192.168.64.129|Bind ip|
|protected-mode|yes|开启保护模式|
|port|6381|redis端口号|
|daemonize|yes|守护进程启动|
|pidfile|/var/run/redis_6381.pid|pid文件|
|dir|/usr/local/redis-4.0.1/conf/cluster/6380|数据存放目录|
|databases|1|默认为16，集群时设置一个即可|
|cluster-enabled|yes|开启集群|
|cluster-config-file|/usr/local/redis-4.0.1/config/cluster/6381/nodes.conf|集群配置文件（启动时自动生成在当前目录下，建议在6379目录下启动redis节点）|
|cluster-node-timeout|15000|节点互联超时时间，毫秒|
|cluster-migration-barrier|1|数据迁移的副本临界数，这个参数表示一个主节点在拥有多少个从节点时就要割让一个从节点给另一个没有任何从节点的主节点|

分别在每台服务器上启动redis服务：
```
[root@sjq-02 6379]# /usr/local/redis-4.0.1/bin/redis-server ./redis.conf
[root@sjq-02 6380]# /usr/local/redis-4.0.1/bin/redis-server ./redis.conf
[root@sjq-02 6381]# /usr/local/redis-4.0.1/bin/redis-server ./redis.conf

```

可以使用 <font color='red'>``ps aux|grep redis``</font> 查看服务是否正常启动。

执行Redis 集群创建命令（<font color='red'>**只需要在其中一个节点上执行一次即可**</font>）
```
[root@sjq-02 ~]$ cd /usr/local/redis-4.0.1/
[root@sjq-02 redis-4.0.1]#  cp /usr/local/redis-4.0.1/src/redis-trib.rb /usr/local/bin/redis-trib
[root@sjq-01 6379]# redis-trib create --replicas 1 192.168.64.128:6379 192.168.64.129:6379 192.168.64.128:6380 192.168.64.129:6380 192.168.64.128:6381  192.168.64.129:6381
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.64.128:6379
192.168.64.129:6379
192.168.64.128:6380
Adding replica 192.168.64.129:6380 to 192.168.64.128:6379
Adding replica 192.168.64.128:6381 to 192.168.64.129:6379
Adding replica 192.168.64.129:6381 to 192.168.64.128:6380
M: f1a1123ca63ed4864a166b34252568ae68b7f79c 192.168.64.128:6379
   slots:0-5460 (5461 slots) master
M: fd8a224b4f220fd4f944fd1b3c743c548ac7eeb3 192.168.64.129:6379
   slots:5461-10922 (5462 slots) master
M: e0a135a0708e9c2ec51dff0080cffe9ba4983fe2 192.168.64.128:6380
   slots:10923-16383 (5461 slots) master
S: c4f8d1ef105b8285132003e0bc6067c5e5195bc4 192.168.64.129:6380
   replicates f1a1123ca63ed4864a166b34252568ae68b7f79c
S: c5861fd86db135dd19bf5b48f7f15343b1844833 192.168.64.128:6381
   replicates fd8a224b4f220fd4f944fd1b3c743c548ac7eeb3
S: 380b1f8cd50ab6f295a8f53ef25f46f424ae8c4d 192.168.64.129:6381
   replicates e0a135a0708e9c2ec51dff0080cffe9ba4983fe2
Can I set the above configuration? (type 'yes' to accept): yes
```

如上所示：主节点(M)分别为: 
192.168.64.128:637 ，哈希槽为(0-5460) 
192.168.64.129:6379 ，哈希槽为(5461-10922)
192.168.64.128:6380 ，哈希槽为(10923-16383)
从节点(S)分别为：192.168.64.129:6380、192.168.64.128:6381、192.168.64.129:6381

创建集群时会自动分配主从节点，如果你觉得上面的主从节点不是你想要的，可以 ctrl + c 退出交互模式，交换上面创建集群命令的ip地址的位置，如果是你想要的，输入 <font color='red'>**yes**</font> ，回车，开始创建集群，信息如下：
```
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join..
>>> Performing Cluster Check (using node 192.168.64.128:6379)
M: f1a1123ca63ed4864a166b34252568ae68b7f79c 192.168.64.128:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: c4f8d1ef105b8285132003e0bc6067c5e5195bc4 192.168.64.129:6380
   slots: (0 slots) slave
   replicates f1a1123ca63ed4864a166b34252568ae68b7f79c
S: 380b1f8cd50ab6f295a8f53ef25f46f424ae8c4d 192.168.64.129:6381
   slots: (0 slots) slave
   replicates e0a135a0708e9c2ec51dff0080cffe9ba4983fe2
M: e0a135a0708e9c2ec51dff0080cffe9ba4983fe2 192.168.64.128:6380
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: fd8a224b4f220fd4f944fd1b3c743c548ac7eeb3 192.168.64.129:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: c5861fd86db135dd19bf5b48f7f15343b1844833 192.168.64.128:6381
   slots: (0 slots) slave
   replicates fd8a224b4f220fd4f944fd1b3c743c548ac7eeb3
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

## 3.2、创建集群过程说明： ##
- 给定的<font color='red'>``redis-trib``</font> 程序的命令为 <font color='red'>``create``</font> ,这表示创建一个新的集群；
- <font color='red'>``--replicas 1``</font> 表示每个主节点下有一个从节点；
- 之后跟着其它参数则是实例的地址列表，程序会使用这些地址的实例创建集群。接着，redis-trib会打印一些预想中的配置给你看，如果你觉得没问题，就可以输入 yes，redis-trib 就会将这份配置应用到集群中。

查看集群状（可以在任意一台节点上查看）：
03.png
从上图可以看出，每个集群的主、从节点、以及每个master节点的哈希槽 redis-cluster把所有的物理节点映射到 【0-16383】个slot(哈希槽)上，cluster负责维护 node<->slot<->value

## 3.3、集群高可用测试： ##
### 3.3.1、重建集群步骤 ###
网上的说法是删除每个节点的nodes.conf、appendonly.aof、dump.rdb文件，再重启每个节点，使用 redis-trib create 命令创建集群，此种方式不能在生产环境中使用，如果删除这些文件, 持久化的数据都不存在，这在生产环境是非常危险的。
正确做法是关闭所有集群节点，直接启动每个节点即可，不要删除nodes.conf 、appendonly.aof 、dump.rdb文件，重启redis后将自动加入到集群中。


### 3.3.2、查看当前集群各节点的状态： ###
04.png

### 3.3.3、redis-trib.rb命令参数说明： ###

- <font color='red'>**call**</font> : 执行redis命令
- <font color='red'>**create**</font>：创建一个新的集群
- <font color='red'>**add-node**</font>:将一个节点添加到集群中，第一个是新节点ip:port ，第二个是任意一个已存在节点 ip:port，如将新节点192.168.64.135:6379添加到集群中：
 redis-trib add-node 192.168.64.135:6379 192.168.64.128:6379
- <font color='red'>**reshard**</font>:重新分片
- <font color='red'>**check**</font>:查看集群信息
- <font color='red'>**del-node**</font>:移除一个节点

# 四、为集群添加新节点： #
在 192.168.64.135上安装 redis，步骤如上，并启动两个redis实例：192.168.64.135:6379（master） 和192.168.64.135:6380（slave）

## 4.1、添加主节点 ##
在任意一台redis集群服务器上使用 add-node 将一个节点添加到集群中
```
/usr/local/redis-4.0.1/src/redis-trib.rb add-node 192.168.64.135:6379 192.168.64.128:6379
```
05.png

执行上面的命令，就会添加新的节点到集群中，新的节点不包含任何数据，因为它没有分配 slot(哈希槽),默认新添加的节点是master(主)节点，当集群需要将某个从节点升级为新的主节点时，这个新节点不会被选中。

为新节点添加slot(哈希槽)：
只需要指定集群中一个节点的地址，redis-trib 就会自动找到集群中的其他节点信息，目前，redis-trib只能在管理员的协助下完成重新分片的工作，命令如下：
```
/usr/local/redis-4.0.1/src/redis-trib.rb reshard 192.168.64.129:6379
```
06.png

上面的会询问你需要移动多少个哈希槽(slot) ，从 1 – 16384，这里输入 500。

07.png

上面又会询问你接收的节点 id是哪个，很显然这个节点id为刚添加的新节点id，可以连接redis集群查看如下：

08.png
新添加的节点为 红色框部分，解读如下：

|参数|描述|
|:-:|:-:|
|63f13eba35c50dc8503bebad513f4265368c7aa4|节点id|
|192.168.64.135:6379@16379|主机ip:端口号@集群链接端口号|
|master|表示为主节点|
|-|如果该节点为master,以 – 表示，如果为 slave（从节点）, 则为该节点的master id|
|connected  0-5460|如果connected后面有数字，表示为主节点的哈希槽，只有master的connected才有可能有哈希槽的值，slave不会有|

很显然，在上面应该输入新增加节点的id 即: 63f13eba35c50dc8503bebad513f4265368c7aa4
09.png

又会询问，需要从哪些节点分配 ，如果输入**all** ，会将每个 master节点分配500个哈希槽到新的节点上，你也可以指定输入想要从哪些节点移动到新节点的master id，然后输入 **done** 完成。这里我输入 **all** 。
然后就给你列出一些移动哈希槽的计划列表，如果没有问题，输入**yes**  即可
10.png

再查看集群节点信息如下：我们可以发现此时 192.168.64.135:6379 节点下有哈希槽了，主节点添加成功。

11.png

## 4.2、添加从节点 ##
先在任意一台中集群节点服务上执行以下命令将新的节点添加到集群中：
```
# /usr/local/redis-4.0.1/src/redis-trib.rb add-node 192.168.64.135:6380 192.168.64.128:6379
```
查看节点信息如下：
12.png

新添加的节点默认为主节点，将新节点设置为192.168.64.135：6379的 从节点，操作如下：
使用redis-cli 连接上新节点的shell，输入命令：<font color='red'>**cluster replicate**</font> 对应master的节点id
```
[root@sjq-09 6379]# /usr/local/redis-4.0.1/bin/redis-cli -c -h 192.168.64.135 -p 6380
192.168.64.135:6380> cluster replicate 63f13eba35c50dc8503bebad513f4265368c7aa4
OK
192.168.64.135:6380>
```
查看集群节点信息如下，此时192.168.64.135:6380(slave)的主节点为192.168.64.135:6379
13.png

注意：
在线添加slave时，需要 dump 整个master进程，并传递到slave，再由slave加载rdb文件到内存，rdb传输过程中master可能无法提供服务，整个过程消耗大量IO,要小心操作，最好在服务器访问量小的时候进行。


# 五、为集群删除节点 #
## 5.1	删除slave节点： ##

使用 <font color='red'>**redis-trib del-node**</font> 节点ip:port 节点id 删除：
删除192.168.64.135:6380节点，删除节点后，也会将该节点服务停用。
14.png

查看集群节点信息如下，可知删除的节点已不在集群中：
15.png


## 5.2、删除master节点： ##
删除主节点之前，需要先使用 reshard移除该 master 的全部slot(哈希槽),然后再删除当前节点，（目前只能将删除的master的slot迁移到一个节点上），操作方式与配置slot类似，指定具体的Source node即可。下面以删除192.168.64.135:6379节点为例说明：
16.png

17.png

18.png

19.png

再次查看节点信息如下，发现 192.168.64.135:6379没有了哈希槽(slot)

20.png

此时，我们就可以删除节点了

21.png

再次查看节点信息，发现 192.168.64.135:6379 这个节点已不在集群中。

22.png

# 六、为集群添加密码授权 #

- 方式一：为集群添加密码授权
在每个redis服务的redis.conf 添加内容如下：
```
masterauth 1234567 
requirepass 1234567
``` 
说明：这种方式需要重新启动各节点

- 方式二： 进入各个实例进行设置
```
./redis-cli -c -h 192.168.64.128 -p 6379 
config set masterauth 1234567 
config set requirepass 1234567 
config rewrite
``` 
注意：各个节点密码都必须一致，否则Redirected就会失败， 推荐这种方式，这种方式会把密码写入到redis.conf里面去，且不用重启。

设置密码之后如果需要使用redis-trib.rb的各种命令，如下：
```
redis-trib.rb check 192.168.64.128，
[ERR] Sorry, can’t connect to node 192.168.64.128:7000
``` 
上面会报错，解决方法：
<font color='red'>vim /usr/local/rvm/gems/ruby-2.3.3/gems/redis-4.0.1/lib/redis/client.rb</font> ,找到password,并修改password为上面配置的密码即可
```
class Redis
  class Client

    DEFAULTS = {
      :url => lambda { ENV["REDIS_URL"] },
      :scheme => "redis",
      :host => "127.0.0.1",
      :port => 6379,
      :path => nil,
      :timeout => 5.0,
      :password => "1234567",
      :db => 0,
      :driver => nil,
      :id => nil,
      :tcp_keepalive => 0,
      :reconnect_attempts => 1,
      :inherit_socket => false
    }

    attr_reader :options

    def scheme
      @options[:scheme]
    end

    def host
      @options[:host]
    end

    def port
      @options[:port]
    end

```






