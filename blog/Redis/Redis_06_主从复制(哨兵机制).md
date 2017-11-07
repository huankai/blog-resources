---
title: Redis 主从复制
date: {{ date }}
author: huangkai
tags: 
	- Redis
---
redis复制的非常重要特性(摘自官网)：
- 一个Master可以有多个Slaves
- Slaves能过接受其他slave的链接，除了可以接受同一个master下面slaves的链接以外，还可以接受同一个结构图中的其他slaves的链接
- redis复制是在master段是非阻塞的，这就意味着master在同一个或多个slave端执行同步的时候还可以接受查询
- 复制在slave端也是非阻塞的，假设你在redis.conf中配置redis这个功能，当slave在执行的新的同步时，它仍可以用旧的数据信息来提供查询，否则，你可以配置当redis slaves去master失去联系时，slave会给发送一个客户端错误
- 为了有多个slaves可以做只读查询，复制可以重复2次，甚至多次，具有可扩展性
- 通过复制可以避免master全量写硬盘的消耗：只要配置 master 的配置文件redis.conf来“避免保存”（注释掉所有”save”命令），然后连接一个用来持久化数据的slave即可。但是这样要确保masters 不会自动重启。


一主两从配置Redis主从复制:服务如下

master ： 192.168.1.90 6379
slaveof : 192.168.1.90 6380
slaveof : 192.168.1.90 6381


## 1、 创建配置文件 ##
创建 redis_6380.conf 与  redis_6381.conf配置文件
```
[root@huangkai conf]# pwd
/usr/local/redis-4.0.1/conf
[root@huangkai conf]# ll
total 60
-rw-r--r--. 1 root root 57777 Oct 29 02:17 redis.conf
[root@huangkai conf]# cp redis.conf redis_6380.conf
[root@huangkai conf]# cp redis.conf redis_6381.conf  
[root@huangkai conf]# ll
total 180
-rw-r--r--. 1 root root 57777 Nov  6 21:44 redis_6380.conf
-rw-r--r--. 1 root root 57777 Nov  6 21:44 redis_6381.conf
-rw-r--r--. 1 root root 57777 Oct 29 02:17 redis.conf
[root@huangkai conf]# 
```
## 2、修改配置文件 ##
redis.conf默认配置不用修改。
 
redis_6380.conf修改如下：
```
...
port 6380
pidfile /var/run/redis_6380.pid
logfile "redis_6380.log"
dbfilename dump_6380.rdb
# appendfilename "appendonly_6380.aof" appendonly持久化未开启，可不修改
... 
```

redis_6381.conf修改如下：
```
...
port 6381
pidfile /var/run/redis_6381.pid
logfile "redis_6381.log"
dbfilename dump_6381.rdb
# appendfilename "appendonly_6381.aof" appendonly持久化未开启，可不修改
... 
```

## 3、启动三台Redis服务 ## 

启动 6379

```
[root@huangkai conf]# redis-server ./redis.conf 
948:C 06 Nov 21:52:07.639 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
948:C 06 Nov 21:52:07.639 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=948, just started
948:C 06 Nov 21:52:07.639 # Configuration loaded
[root@huangkai conf]# netstat -an|grep 6379
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN     
tcp6       0      0 :::6379                 :::*                    LISTEN     
[root@huangkai conf]# 

[root@huangkai ~]# redis-cli -p 6379
127.0.0.1:6379> 
127.0.0.1:6379>
```

启动 6380
```
[root@huangkai conf]# redis-server ./redis_6380.conf 
[root@huangkai conf]# netstat -an|grep 6380
tcp        0      0 0.0.0.0:6380            0.0.0.0:*               LISTEN     
tcp6       0      0 :::6380                 :::*                    LISTEN     
[root@huangkai conf]# 

[root@huangkai ~]# redis-cli -p 6380
127.0.0.1:6380> 
127.0.0.1:6380>
```

启动 6381
```
[root@huangkai conf]# redis-server ./redis_6381.conf 
[root@huangkai conf]# netstat -an|grep 6381          
tcp        0      0 0.0.0.0:6381            0.0.0.0:*               LISTEN     
tcp6       0      0 :::6381                 :::*                    LISTEN     
[root@huangkai conf]#

[root@huangkai ~]# redis-cli -p 6381
127.0.0.1:6381> 
127.0.0.1:6381> 
``` 
现在，三台服务已启动，并且都成功连接上，使用 ``INFO REPLICATION`` 命令查看信息如下：

6379：
```
127.0.0.1:6379> INFO REPLICATION
# Replication
role:master
connected_slaves:0
master_replid:d7f13a179d3808311c9dc4467a7a68619d3bf5b8
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6379> 
```

6380：
```
127.0.0.1:6380> INFO REPLICATION
# Replication
role:master
connected_slaves:0
master_replid:564db58a73b1dd19d262bd823bd590faaacec50e
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6380> 
```

6381：
```
127.0.0.1:6381> INFO REPLICATION
# Replication
role:master
connected_slaves:0
master_replid:24a3159635d19972c822f86f024dc03fbad9c65d
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6381> 
```
从上面的信息可以看出，三台服务的 角色都 为 master，连接的 slaves为0，也就是这三台服务现在还没有任何关系，在任意一台服务中设置值，其它两台都不会同步数据。
如：在 6379 设置 k1 为 k1 ，如下

```
127.0.0.1:6379> SET k1 k1
OK
127.0.0.1:6379> get k1
"k1"
127.0.0.1:6379> 
```

6380 获取k1的值，返回无结果，如下：
```
127.0.0.1:6380> get k1
(nil)
127.0.0.1:6380> 
```

6381 获取k1的值，返回也无结果，如下：
```
127.0.0.1:6381> get k1
(nil)
127.0.0.1:6381>
```

## 4、设置主从 ##
### 4.1、最简单的主从配置 ###


现在将  6379设置为master， 6380 与 6381 设置为slave，配置如下：

6380：

```
127.0.0.1:6380> SLAVEOF 192.168.1.90 6379
OK
127.0.0.1:6380>
```


6381：

```
127.0.0.1:6381> SLAVEOF 192.168.1.90 6379
OK
127.0.0.1:6381> 
```

查看各服务信息：

6379：
由下可知： 此服务角色为 master，现在有两个连接的 slave,分别为 192.168.1.90:6380 与 192.168.1.90:6381,并且都为在线状态。
```
127.0.0.1:6379> INFO REPLICATION
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.1.90,port=6380,state=online,offset=112,lag=1
slave1:ip=192.168.1.90,port=6381,state=online,offset=112,lag=0
master_replid:7499fc3cfd327ecb9bf72c79e53cb5014f14230c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:112
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:112
127.0.0.1:6379> 
```

6380：
由下可以，此时此服务的角色已变为 slave，它的master 主机为 192.168.1.90，端口号为 6379。
```
127.0.0.1:6380> INFO REPLICATION
# Replication
role:slave
master_host:192.168.1.90
master_port:6379
master_link_status:up
master_last_io_seconds_ago:6
master_sync_in_progress:0
slave_repl_offset:140
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:7499fc3cfd327ecb9bf72c79e53cb5014f14230c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:140
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:140
127.0.0.1:6380>
```

6381：
由下可以，此时此服务的角色也变为了 slave，它的master 主机为 192.168.1.90，端口号为 6379。
```
127.0.0.1:6381> INFO REPLICATION
# Replication
role:slave
master_host:192.168.1.90
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:168
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:7499fc3cfd327ecb9bf72c79e53cb5014f14230c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:168
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:154
127.0.0.1:6381>
```

之前在还没有配置slave时，我们在6379服务中配置了 k1 值，此时主从复制配置成功后， 6380与 6381是否会有 6379中配置的值呢？
使用命令查看一下：
```
127.0.0.1:6380> get k1
"k1"
127.0.0.1:6380> 
```

```
127.0.0.1:6381> get k1
"k1"
127.0.0.1:6381> 
```
如上，可以看到 6380 与 6381 都能获取到值。
**结论一：配置主从后，从服务器(slave)会复制主服务(master)的所有值。**

### 4.2、从服务器设置值 ###
现在，我们在slave服务器上设置值，看看会出现什么情况 ：

6380:
```
127.0.0.1:6380> set k2 k2
(error) READONLY You can't write against a read only slave.
127.0.0.1:6380> 
```
6381:
```
127.0.0.1:6381> set k3 v3
(error) READONLY You can't write against a read only slave.
127.0.0.1:6381>
```
如上，在从服务器上设置值，抛出了一个错误，意思是 slave只能读取数据，不能写数据。
**结论二：配置主从后，所有的写操作都只能在主服务器(master)进行，从服务器(slave)只能读数据不能写数据。如果需要在从服务器中支持写功能，可以在需要写的slave执行命令 ``config set slave-read-only no`` ，也就是使当前slave支持写功能。但是此种配置后，写入的数据只在当前slave中存在，不会同步其它的slave,更不会同步到master。 **


### 4.3、主服务器宕机 ###

现在，我们将主服务器(master(6379))停止服务，查看变化
```
127.0.0.1:6379> SHUTDOWN
not connected> 
```
查看 6380 如下：
```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:192.168.184.128
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:854
master_link_down_since_seconds:26
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:ca0c52c31eefe3567c0beb099b83e174080e054e
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:854
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:854
127.0.0.1:6380> 
```
查看 6381 如下：
```
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:192.168.184.128
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:854
master_link_down_since_seconds:31
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:ca0c52c31eefe3567c0beb099b83e174080e054e
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:854
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:840
127.0.0.1:6381> 
```
可以发现，slave并没有进行选举产生出master，现在它们的角色依然还是 slave。

**结论三：当从服务器``SHUTDOWN``后，slave并没有通过选举产生新的master，此时会导致整个服务不可写。所以，在使用此种方式时，一定要将 master持久化，如果 master通过一个空数据集重启，slave中的数据也将被清空。**

### 4.4、主服务器正常运行 ###

现在启动 master，查看变化
```
not connected> quit
[root@huangkai ~]# cd /usr/local/redis-4.0.1/conf/
[root@huangkai conf]# redis-server ./redis.conf 
2196:C 07 Nov 11:44:58.813 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
2196:C 07 Nov 11:44:58.813 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=2196, just started
2196:C 07 Nov 11:44:58.813 # Configuration loaded
[root@huangkai conf]# 
```

连接 6379，查看信息如下：
```
[root@huangkai conf]# redis-cli -p 6379
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=910,lag=0
slave1:ip=127.0.0.1,port=6380,state=online,offset=910,lag=0
master_replid:3fe903a1fab24acafb159f837c6bb1b7e1d423a5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:910
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:855
repl_backlog_histlen:56
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> 
```

6380 信息如下：
```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:192.168.184.128
master_port:6379
master_link_status:up
master_last_io_seconds_ago:5
master_sync_in_progress:0
slave_repl_offset:924
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:3fe903a1fab24acafb159f837c6bb1b7e1d423a5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:924
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:855
repl_backlog_histlen:70
127.0.0.1:6380> get k2
"v2"
127.0.0.1:6380>

```

6381 信息如下：
```
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:192.168.184.128
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:938
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:3fe903a1fab24acafb159f837c6bb1b7e1d423a5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:938
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:855
repl_backlog_histlen:84
127.0.0.1:6381> get k2
"v2"
127.0.0.1:6381>
```
通过以上信息可知，当 master 重新恢复后，整个服务都可正常使用。


### 4.5、既主既从 ###

现在将 6380 设置为 6379的 slave ，并且设置为 6381的 master

6379 不需要修改配置

将6380设置为6379的slave如下：
```
127.0.0.1:6380> SLAVEOF 162.168.184.128 6379
OK
127.0.0.1:6380> 
```

将6381设置为6380的 slave 如下：
```
127.0.0.1:6381> SLAVEOF 162.168.184.128 6380
OK
127.0.0.1:6381> 
```

查看 6379信息如下：
```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=9180,lag=1
master_replid:3fe903a1fab24acafb159f837c6bb1b7e1d423a5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:9180
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:855
repl_backlog_histlen:8326
127.0.0.1:6379> 
```
查看 6380信息如下：
```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:192.168.184.128
master_port:6379
master_link_status:up
master_last_io_seconds_ago:9
master_sync_in_progress:0
slave_repl_offset:9180
slave_priority:100
slave_read_only:0
connected_slaves:0
master_replid:3fe903a1fab24acafb159f837c6bb1b7e1d423a5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:9180
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:855
repl_backlog_histlen:8326
127.0.0.1:6380> 
```
查看 6381 信息如下：
```
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:162.168.184.128
master_port:6380
master_link_status:up
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:9166
master_link_down_since_seconds:1510032310
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:3fe903a1fab24acafb159f837c6bb1b7e1d423a5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:9166
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:855
repl_backlog_histlen:8312
127.0.0.1:6381> 
```
通过以上可知 ：
6379 角色为 master，slave 只有一个，为 6380
6380 角色为 slave，master 为 6379
6380 角色为 slave，master 为 6380

6379 设置值 ：
```
127.0.0.1:6379> set k3 v3
OK
127.0.0.1:6379> 
```

6380 获取值 ：
```
127.0.0.1:6380> get k3
"v3"
127.0.0.1:6380> 
```

6381 获取值 ：
```
127.0.0.1:6381> get k3
"v3"
127.0.0.1:6381>
```

由上可知，此种方式配置，也会同步 6381 服务的数据。

### 4.6、反从为主 ###



### 4.7、redis哨兵模式(sentinel) ###

Redis 的 Sentinel 用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务：
- 监控（Monitoring）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
- 提醒（Notification）： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- 自动故障迁移（Automatic failover）： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

可以这样说，Redis的sentinel就是 [反从为主](#4.6-)的自动档。

#### 4.7.1、sentinel 配置文件： ####

安装完redis后，在redis根目录下，会有sentinel.conf 配置文件，该配置文件参数如下：

```
#配置sentinel端口号，默认为 26379
port 26379 
--------------------------------------------------
#sentinel目录
dir /tmp
--------------------------------------------------
# 告诉sentinel去监听地址为ip:port的一个master，这里的master-name可以自定义，quorum是一个数字，
#指明当有多少个sentinel认为一个master失效时，master才算真正失效。
#master-name只能包含英文字母，数字和“.-_”这三种字符,需要注意的是master-ip 要写真实的ip地址而不要用回环地址（127.0.0.1）
#配置示例：sentinel monitor mymaster 192.168.0.5 6379 1
sentinel monitor <master-name> <ip> <redis-port> <quorum>
--------------------------------------------------
#设置连接master和slave时的密码，注意的是sentinel不能分别为master和slave设置不同的密码，因此master和slave的密码应该设置相同。
配置示例：sentinel auth-pass mymaster 0123passw0rd
sentinel auth-pass <master-name> <password>
--------------------------------------------------
#指定需要多少失效时间，一个master才会被这个sentinel主观地认为是不可用的。 单位是毫秒，默认为30秒
#配置示例： sentinel down-after-milliseconds mymaster 30000
sentinel down-after-milliseconds <master-name> <milliseconds> 
--------------------------------------------------
#指定在发生failover(主备切换时)最多可以有多少个slave同时对新的master进行同步，这个数字越小，完成failover所需的时间就越长，但是如果这个数字越大，就意味着越多的slave因为replication而不可用。可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
#配置示例： sentinel parallel-syncs mymaster 1
sentinel parallel-syncs <master-name> <numslaves> 
--------------------------------------------------
#failover-timeout 可以用在以下这些方面：
1. 同一个sentinel对同一个master两次failover之间的间隔时间。
2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
3. 当想要取消一个正在进行的failover所需要的时间。  
4. 当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了。
配置示例：sentinel failover-timeout mymaster1 20000
sentinel failover-timeout <master-name> <milliseconds>
--------------------------------------------------
sentinel的notification-script和reconfig-script是用来配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。对于脚本的运行结果有以下规则：
1.若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
2.若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。

sentinel notification-script <master-name> <script-path> 
通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，一个是事件的类型，一个是事件的描述。如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
配置示例：sentinel notification-script mymaster /var/redis/notify.sh

sentinel client-reconfig-script <master-name> <script-path>
当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。以下参数将会在调用脚本时传给脚本:
<master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
目前<state>总是“failover”, <role>是“leader”或者“observer”中的一个。 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的。这个脚本应该是通用的，能被多次调用，不是针对性的。
配置示例：sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```

#### 4.7.2、一主双从三sentinel ####
通过配置一主双从三sentinel实现redis 监控与自动故障迁移。

ip地址分配如下:

master:192.168.1.90 6379
slave1:192.168.1.90 6380
slave2:192.168.1.90 6381
sentinel1 : 192.168.1.90 26379
sentinel2 : 192.168.1.90 26389
sentinel3 : 192.168.1.90 26399

一主三从配置见 [修改配置文件](#2-)

sentinel配置如下：

sentinel1:
```
#将配置文件复制到conf目录，保留出厂配置
[root@huangkai redis-4.0.1]#cp sentinel.conf conf/

#修改配置文件，内容如下

port 26379 #端口号，
daemonize yes # 是否以守护进程启动，默认此文件中没有此配置，默认值为 no，如果不添加，启动sentinel时，进程会直接在前台跑，一退出sentinel进程就关了，还一种方式是以 nohub 来启动
logfile "/var/log/sentinel_26379.log"  #日志文件
```

sentinel2:
```
[root@huangkai conf]# cp sentinel.conf sentinel_26389.conf 
[root@huangkai conf]# 
#修改配置文件，内容如下
port 26389 #端口号，
daemonize yes
logfile "/var/log/sentinel_26389.log" 
```

sentinel3:
```
[root@huangkai conf]# cp sentinel.conf sentinel_26399.conf 
[root@huangkai conf]# 
#修改配置文件，内容如下
port 26399 #端口号，
daemonize yes
logfile "/var/log/sentinel_26399.log" 
```

启动 sentinel1
```
[root@huangkai conf]# redis-sentinel ./sentinel.conf 
[root@huangkai conf]# 

#连接
[root@huangkai ~]# redis-cli -p 26379
127.0.0.1:26379> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "127.0.0.1"
 5) "port"
 6) "6379"
 7) "runid"
 8) "334ab6dba2c2e236929d925fc21298649736bcae"
 9) "flags"
10) "master"
11) "link-pending-commands"
12) "0"
13) "link-refcount"
14) "1"
15) "last-ping-sent"
16) "0"
17) "last-ok-ping-reply"
18) "876"
19) "last-ping-reply"
20) "876"
21) "down-after-milliseconds"
22) "30000"
23) "info-refresh"
24) "1992"
25) "role-reported"
26) "master"
27) "role-reported-time"
28) "674576"
29) "config-epoch"
30) "0"
31) "num-slaves"
32) "2"
33) "num-other-sentinels"
34) "0"
35) "quorum"
36) "2"
37) "failover-timeout"
38) "180000"
39) "parallel-syncs"
40) "1"
127.0.0.1:26379> 
```

启动 sentinel2
```
[root@huangkai conf]# redis-sentinel ./sentinel_26389.conf 

#连接
[root@huangkai ~]# redis-cli -p 26389
127.0.0.1:26389> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "127.0.0.1"
 5) "port"
 6) "6379"
 7) "runid"
 8) "334ab6dba2c2e236929d925fc21298649736bcae"
 9) "flags"
10) "master"
11) "link-pending-commands"
12) "0"
13) "link-refcount"
14) "1"
15) "last-ping-sent"
16) "0"
17) "last-ok-ping-reply"
18) "122"
19) "last-ping-reply"
20) "122"
21) "down-after-milliseconds"
22) "30000"
23) "info-refresh"
24) "1972"
25) "role-reported"
26) "master"
27) "role-reported-time"
28) "122532"
29) "config-epoch"
30) "0"
31) "num-slaves"
32) "2"
33) "num-other-sentinels"
34) "0"
35) "quorum"
36) "2"
37) "failover-timeout"
38) "180000"
39) "parallel-syncs"
40) "1"
127.0.0.1:26389>  
```

启动 sentinel3
```
[root@huangkai conf]# redis-sentinel ./sentinel_26399.conf 

#连接
[root@huangkai ~]# redis-cli -p 26399
127.0.0.1:26399> sentinel master mymaster
 1) "name"
 2) "mymaster"
 3) "ip"
 4) "127.0.0.1"
 5) "port"
 6) "6379"
 7) "runid"
 8) "334ab6dba2c2e236929d925fc21298649736bcae"
 9) "flags"
10) "master"
11) "link-pending-commands"
12) "0"
13) "link-refcount"
14) "1"
15) "last-ping-sent"
16) "0"
17) "last-ok-ping-reply"
18) "442"
19) "last-ping-reply"
20) "442"
21) "down-after-milliseconds"
22) "30000"
23) "info-refresh"
24) "9464"
25) "role-reported"
26) "master"
27) "role-reported-time"
28) "119978"
29) "config-epoch"
30) "0"
31) "num-slaves"
32) "2"
33) "num-other-sentinels"
34) "0"
35) "quorum"
36) "2"
37) "failover-timeout"
38) "180000"
39) "parallel-syncs"
40) "1"
127.0.0.1:26399> 
```

到此，一主双从三sentinel配置完成。

停止 6379(master)，查看变化:
```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=133321,lag=1
slave1:ip=127.0.0.1,port=6380,state=online,offset=133454,lag=1
master_replid:3fe903a1fab24acafb159f837c6bb1b7e1d423a5
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:133587
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:855
repl_backlog_histlen:132733
127.0.0.1:6379> SHUTDOWN
not connected> 
```


主从复制的缺点：
由于所有的写操作都是在master上进行，然后同步到slave，master复制到 slave会有一定的延时，当系统繁忙时，可能延时会更加严重，slave机器增多也会使这个问题更严重。 







