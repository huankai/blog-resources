---
title: Redis配置文件 
date: {{ date }}
author: huangkai
tags:
    - Redis
---

Redis 的主配置文件在 Redis安装根目录下redis.conf，文件详细配置如下：

配置文件单位说明：
- 1k => 1000 bytes
- 1kb => 1024 bytes
- 1m => 1000000 bytes
- 1mb => 1024*1024 bytes
- 1g => 1000000000 bytes
- 1gb => 1024*1024*1024 bytes
如上可知， 1g 与 1gb所表示的大小是有区别的，单位大小写不敏感，1gb = 1Gb = 1gB = 1GB

# 1、includes #
载入其它配置文件信息
比如说当你有多个server，而有一些配置项是它们公用的，那么你可以将这些公用的配置项写进一个配置文件common.conf里，然后这些server再include这个配置文件，这些server自己的配置项则分别写在自己的配置文件里。
示例：include /path/to/common.conf
# 2、modules #
模块系统，是Redis 4.0新功能之一。
这个系统可以让用户通过自己编写的代码来扩展和实现 Redis 本身并不具备的功能， 具体使用方法可以参考 antirez 的博文《Redis Loadable Module System》： [http://antirez.com/news/106](http://antirez.com/news/106)

因为模块系统是通过高层次 API 实现的， 它与 Redis 内核本身完全分离、互不干扰， 所以用户可以在有需要的情况下才启用这个功能， 以下是 redis.conf 中记载的模块载入方法：
```
################################## MODULES #####################################

# Load modules at startup. If the server is not able to load modules
# it will abort. It is possible to use multiple loadmodule directives.
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so
```
目前已经有人使用这个功能开发了各种各样的模块， 比如 Redis Labs 开发的一些模块就可以在 http://redismodules.com 看到， 此外 antirez 自己也使用这个功能开发了一个神经网络模块： https://github.com/antirez/neural-redis
模块功能使得用户可以将 Redis 用作基础设施， 并在上面构建更多功能， 这给 Redis 带来了无数新的可能性。
# 3、network #

```
################################## NETWORK #####################################
bind 127.0.0.1 192.168.1.90
protected-mode yes
port 6379
tcp-backlog 511
# unixsocket /tmp/redis.sock
# unixsocketperm 700
timeout 0
tcp-keepalive 300
```
## 3.1、bind ##
默认情况下,bind接口是127.0.0.1，也就是本地回环地址，这样，Redis只能通过本机客户端连接，而无法通过远程连接，这样可以避免将redis服务暴露于危险的网络环境中，防止一些不安全的人随随便便通过远程
连接到redis服务。
如果bind选项为空的话，并且 protected-mode 为 no，会接受所有来自于可用网络接口的连接。
如果bind选项为空的话，并且 protected-mode 为 yes，会选择默认的连接，即只能通过 127.0.0.1连接
语法如下：

```
bind ip1 ip2 ...
```
如果redis安装服务器ip为  192.168.1.90 ，在 192.168.1.4机器 上需要连接 redis服务，那么 bind配置应该如下 
```
#注意，后面的ip并不是配置 192.168.1.4，应该配置redis可接受的ip 192.168.1.90
bind 127.0.0.1 192.168.1.90
protected-mode yes
```
或者如下：
```
protected-mode no
```
## 3.2、protected-mode ##
保护模式：默认为 yes,只接受指定 bind的连接，如果设置为 no ，bind的ip不会生效，接受所有可用的网络连接。生产环境严格建议设置为 yes

## 3.3、port ##
redis配置服务的端口号参数

## 3.4、timeout ##
客户端连接超时时间，单位为秒，0 为禁止

# 4、GENERAL #
```
################################# GENERAL #####################################
daemonize yes
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
syslog-enabled no
syslog-ident redis
syslog-facility local0
databases 16
```
## 4.1、daemonize ##
redis是否以守护进程启动，默认为 no，可选值  yes | no
## 4.2、supervised ##
可以通过upstart和systemd管理Redis守护进程，这个参数是和具体的操作系统相关的
## 4.3、pidfile ##
redis启动后的pid文件目录，默认为 /var/run/redis_6379.pid
## 4.4、loglevel ##
日志级别：有四个等级 ：debug < verbose < notice < warning ,默认为  notice
## 4.5、logfile ##
日志文件目录，当指定为空字符串时，为标准输出，如果redis已守护进程模式运行，那么日志将会输出到  /dev/null
## 4.6、 syslog-enabled ##
是否把日志记录到系统日志。默认为 no
## 4.7、syslog-ident ##
设置系统日志的id
## 4.8、syslog-facility ##
指定syslog设备(facility)，必须是user或则local0到local7
## 4.9、databases ##
redis database 个数，默认为 16 个，从 0 开始 ，到 15

# 5、SNAPSHOTTING #
快照配置
```
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
```
## 5.1、save ##
rdb持久化
每隔 多少 秒内有 多少个写操作，执行持久化操作,可以配置多个save ,
如果要禁rdb，这里的三个 save都注释就可以了.
## 5.2、stop-writes-on-bgsave-error ##
rdb持久化,redis会在持久化的时候，开启一个新的进程进行持久化，如果在持久化的时候出错，如磁盘空间不足，为保证数据的完整性，此参数控制持久化出错时停止在内存写入数据。默认值就好。
## 5.3、rdbcompression ##
rdb 持久化时是否压缩，默认yes
## 5.4、rdbchecksum ##
重启redis时，检查 rdb文件是否有损坏，默认 yes
## 5.5、dbfilename ##
存的 rdb文件名，默认为 dump.rdb
## 5.6、dir ##
导出的rdb文件放在哪个目录，默认为当前目录

# 6、REPLICATION #
主从复制
```
slaveof <masterip> <masterport>
masterauth <master-password>
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-ping-slave-period 10
repl-timeout 60
repl-disable-tcp-nodelay no
repl-backlog-size 1mb
repl-backlog-ttl 3600
slave-priority 100
min-slaves-to-write 3
min-slaves-max-lag 10
slave-announce-ip 5.5.5.5
slave-announce-port 1234 
```

## 6.1、slaveof ##
设置本机为slave服务。格式：slaveof <masterip> <masterport>。设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步。
```
slave 192.168.1.90 6378
```
## 6.2、masterauth ##
当master服务设置了密码保护时，slav服务连接master的密码。

## 6.3、slave-serve-stale-data ##
当一个slave与master失去联系时，或者复制正在进行的时候，slave应对请求的行为：1) 如果为 yes（默认值） ，slave 仍然会应答客户端请求，但返回的数据可能是过时，或者数据可能是空的在第一次同步的时候；2) 如果为 no ，在你执行除了 info 和 salveof 之外的其他命令时，slave 都将返回一个 "SYNC with master in progress" 的错误。

## 6.4、slave-read-only ##
设置slave是否是只读的。从2.6版起，slave默认是只读的。
## 6.5、repl-diskless-sync ##
主从数据复制是否使用无硬盘复制功能。

## 6.6、repl-diskless-sync-delay ##
当启用磁盘复制时，可以配置延迟时间，以秒为单位，默认为 5 秒，如果设置为 0 ，表示禁用。

## 6.7、repl-ping-slave-period ##
指定slave定期ping master的周期，默认10秒钟。

## 6.8、repl-timeout ##
设置主库批量数据传输时间或者ping回复时间间隔，默认值是60秒 。

## 6.9、repl-disable-tcp-nodelay ##
指定向slave同步数据时，是否禁用socket的NO_DELAY选项。若配置为“yes”，则禁用NO_DELAY，则TCP协议栈会合并小包统一发送，这样可以减少主从节点间的包数量并节省带宽，但会增加数据同步到 slave的时间。若配置为“no”，表明启用NO_DELAY，则TCP协议栈不会延迟小包的发送时机，这样数据同步的延时会减少，但需要更大的带宽。 通常情况下，应该配置为no以降低同步延时，但在主从节点间网络负载已经很高的情况下，可以配置为yes

## 6.10、repl-backlog-size ##

设置主从复制backlog容量大小。这个 backlog 是一个用来在 slaves 被断开连接时存放 slave 数据的 buffer，所以当一个 slave 想要重新连接，通常不希望全部重新同步，只是部分同步就够了，仅仅传递 slave 在断开连接时丢失的这部分数据。这个值越大，salve 可以断开连接的时间就越长。

## 6.11、repl-backlog-ttl ##
配置当master和slave失去联系多少秒之后，清空backlog释放空间。当配置成0时，表示永远不清空。

## 6.12、slave-priority ##
当 master 不能正常工作的时候，Redis Sentinel 会从 slaves 中选出一个新的 master，这个值越小，就越会被优先选中，但是如果是 0 ， 那是意味着这个 slave 不可能被选中。 默认优先级为 100。

## 6.13、min-slaves-to-write ##
首先在本机启动两台redis服务 
192.168.1.90 6379(master)
192.168.1.90 6378(slave)
```
[root@huangkai ~]# redis-cli 
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379> INFO replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6378,state=online,offset=182,lag=1
master_replid:5af7d4dbd0be4052451940ed582c66244dee38b3
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:182
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:182
127.0.0.1:6379> 
```
在master服务中使用info查看信息如上：
可以看到有一个slave,端口号为 6378，在一般情况下，lag的值应该在0秒或者1秒之间跳动，如果超过1秒的话，那么说明主从服务器之间的连接出现了故障。

Redis的min-slaves-to-write和min-slaves-max-lag两个选项可以防止主服务器在不安全的情况下执行写命令
```
min-slaves-to-write 3
min-slaves-max-lag 10
```
那么在从服务器的数量少于3个，或者三个从服务器的延迟（lag）值都大于或等于10秒时，主服务器将拒绝执行写命令，这里的延迟值就是上面提到的INFO replication命令的lag值。
默认情况下 min-slaves-to-write 值为 0 (禁用) ，min-slaves-max-lag 值为 10.
## 6.14、min-slaves-max-lag ##
见 6.13

## 6.15、slave-announce-ip ##
Redis master能够以不同的方式列出所连接slave的地址和端口。 
例如，“INFO replication”部分提供此信息，除了其他工具之外，Redis Sentinel还使用该信息来发现slave实例。
此信息可用的另一个地方在masterser的“ROLE”命令的输出中。
通常由slave报告的列出的IP和地址,通过以下方式获得：
IP：通过检查slave与master连接使用的套接字的对等体地址自动检测地址。
端口：端口在复制握手期间由slavet通信，并且通常是slave正在使用列出连接的端口。
然而，当使用端口转发或网络地址转换（NAT）时，slave实际上可以通过(不同的IP和端口对)来到达。 slave可以使用以下两个选项，以便向master报告一组特定的IP和端口，
以便INFO和ROLE将报告这些值。
如果你需要仅覆盖端口或IP地址，则没必要使用这两个选项。
## 6.16、slave-announce-port ## 
见 6.15

# 7、SECURITY #
安全配置
```
requirepass foobared
rename-command CONFIG ""
```
## 7.1、requirepass ##
设置redis连接密码，生产环境强烈建议设置密码。

## 7.2、rename-command ##
将命令重命名。为了安全考虑，可以将某些重要的、危险的命令重命名。当你把某个命令重命名成空字符串的时候就等于取消了这个命令。

# 8、CLIENTS #
客户端配置
```
maxclients 10000
```
此类只有一个参数，maxclients ,设置客户端最大并发连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数-32（redis server自身会使用一些），如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息。

# 9、 MEMORY MANAGEMENT #
内存管理
```
 maxmemory <bytes>
 maxmemory-policy noeviction
 maxmemory-samples 5
```

## 9.1、maxmemory ##
指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区，格式：maxmemory <bytes>　。
```
maxmemory　1024000
```
## 9.2、maxmemory-policy ##

当内存使用达到最大值时，redis使用的清除策略。有以下几种可以选择：

- volatile-lru -> 利用LRU算法移除设置过过期时间的key (LRU:最近最少使用 Least Recently Used ,也就是首先淘汰最长时间未被使用的key) 
- allkeys-lru -> 利用LRU算法移除任何key
- volatile-lfu ->  利用LFU算法移除设置过过期时间的key(LFU:最近最不常用 Least Frequently Used,也就是淘汰一定时期内被访问次数最少的 key)
- allkeys-lfu -> 利用LFU算法移除任何key
- volatile-random -> 移除设置过过期时间的随机key 
- allkeys-random -> 移除随机key 
- volatile-ttl -> 移除即将过期的key(minor TTL) 
- noeviction -> 不移除任何key，只是返回一个写错误 。默认选项

## 9.3、maxmemory-samples ##
LRU、LFU 和 minimal TTL 算法都不是精准的算法，但是相对精确的算法(为了节省内存)，随意你可以选择样本大小进行检测。redis默认选择5个样本进行检测，你可以通过maxmemory-samples进行设置 样本数

# 10、 LAZY FREEING #

```
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
slave-lazy-flush no
```
# 11、APPEND ONLY MODE #
AOF 持久化，将每执行的命令写入文件中，默认为 no，可以和rdb一起使用
使用 aof有一个问题：当对同一个key多次更改时，其实我们只需要最后一次更新的值，之前执行的命令也会保存起来，这样会导致aof文件过大。
```
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble no
```

## 11.1、appendonly ##
是否启用aof持久化方式 。即是否在每次更新操作后进行日志记录，默认配置是no，即在采用异步方式把数据写入到磁盘。可选值  no | yes

## 11.2、appendfilename ##
aof持久化文件名，默认为 appendonly.aof
## 11.3、appendfsync ##
aof文件刷新的频率。有三种：
- always ：在每个命令都会同步到 aof文件中，安全，速度慢，因为要经常操作磁盘;
- everysec : 择中方案，每秒写一次，最多也只会丢失1秒的数据，默认值;
-  no     :   由操作系统判断缓存大小写入到磁盘,同步频率低，速度快。

## 11.4、no-appendfsync-on-rewrite ##
正在导出 rdb 快照的过程中要不要停止同步aof 默认为 no，默认值即可

## 11.5、auto-aof-rewrite-percentage ##
aof重写参数
对于使用aof持久化时，操作同一个Key导致aof文件过大的问题，可以使用auto-aof-rewrite-percentage 与 auto-aof-rewrite-min-size 两个参数来设定。

aof 文件大小比上次重写时的大小，增长率达到  100%时重写
```
auto-aof-rewrite-percentage 100
```
## 11.6、auto-aof-rewrite-min-size ##

aof文件会从 0 M 增长，前期 aof会进行大量的重写，在此时间没有必须这么频繁的重写，这里指定aof文件超过 64M时才重写，
和上面的 auto-aof-rewrite-percentage 参数是并的关系
```
auto-aof-rewrite-min-size 64mb
```

## 11.7、aof-load-truncated ##
redis在启动时可以加载被截断的AOF文件，而不需要先执行 redis-check-aof 工具
```
aof-load-truncated yes
```
## 11.8、aof-use-rdb-preamble ##
Redis 4.0 新增了 RDB-AOF 混合持久化格式
这是一个可选的功能， 在开启了这个功能之后， AOF 重写产生的文件将同时包含 RDB 格式的内容和 AOF 格式的内容， 其中 RDB 格式的内容用于记录已有的数据， 而 AOF 格式的内存则用于记录最近发生了变化的数据， 这样 Redis 就可以同时兼有 RDB 持久化和 AOF 持久化的优点 —— 既能够快速地生成重写文件， 也能够在出现问题时， 快速地载入数据。
这个功能可以通过此参数进行开启,默认为 no

# 12、 LUA SCRIPTING  #
LUA脚本
```
lua-time-limit 5000
```
此类只有一个参数 lua-time-limit,一个Lua脚本最长的执行时间，单位为毫秒，如果为0或负数表示无限执行时间，默认为5000。

# 13、REDIS CLUSTER #
Redis集群配置
```
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
cluster-slave-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage yes
```
## 13.1、cluster-enabled ##
集群开关，默认不开启集群模式

## 13.2、cluster-config-file ##
集群配置文件的名称，每个节点都有一个集群相关的配置文件，持久化保存集群的信息。
这个文件并不需要手动配置，这个配置文件有Redis生成并更新，每个Redis集群节点需要一个单独的配置文件
请确保与实例运行的系统中配置文件名称不冲突
```
cluster-config-file nodes-6379.conf
```
## 13.3、cluster-node-timeout ##
节点互连超时的阀值，集群节点超时毫秒数

## 13.4、cluster-slave-validity-factor ##
在进行故障转移的时候，全部slave都会请求申请为master，但是有些slave可能与master断开连接一段时间了，
导致数据过于陈旧，这样的slave不应该被提升为master。该参数就是用来判断slave节点与master断线的时间是否过长。判断方法是：
比较slave断开连接的时间和(node-timeout * slave-validity-factor) + repl-ping-slave-period
,如果节点超时时间为三十秒, 并且slave-validity-factor为10,
,假设默认的repl-ping-slave-period是10秒，即如果超过310秒slave将不会尝试进行故障转移
## 13.5、cluster-migration-barrier ##
master的slave数量大于该值，slave才能迁移到其他孤立master上，如这个参数若被设为2，
那么只有当一个主节点拥有2 个可工作的从节点时，它的一个从节点会尝试迁移。
## 13.6、cluster-require-full-coverage ##
默认情况下，集群全部的slot有节点负责，集群状态才为ok，才能提供服务。
设置为no，可以在slot没有全部分配的时候提供服务。
不建议打开该配置，这样会造成分区的时候，小分区的master一直在接受写请求，而造成很长时间数据不一致

# 14、 CLUSTER DOCKER/NAT support #
兼容 NAT 和 Docker，4.0 新功能
```
cluster-announce-ip 10.1.1.5
cluster-announce-port 6379
cluster-announce-bus-port 6380
```

# 15、SLOW LOG #
slowlog是redis用于记录记录慢查询执行时间的日志系统。由于slowlog只保存在内存中，因此slowlog的效率很高，完全不用担心会影响到redis的性能。
```
slowlog-log-slower-than 10000
slowlog-max-len 128
```
## 15.1、slowlog-log-slower-than ##
slowlog的划定界限，只有query执行时间大于slowlog-log-slower-than的才会定义成慢查询，才会被slowlog进行记录。slowlog-log-slower-than设置的单位是微妙，默认是10000微妙，也就是10ms

## 15.2、slowlog-max-len ##
慢查询最大的条数，当slowlog超过设定的最大值后，会将最早的slowlog删除，是个FIFO队列
 
# 16、LATENCY MONITOR #
延迟监控
延迟监控功能是用来监控redis中执行比较缓慢的一些操作，用LATENCY打印redis实例在跑命令时的耗时图表。只有一个参数 latency-monitor-threshold ，只记录大于等于设置的值的操作，0的话，就是关闭监视 
```
latency-monitor-threshold 0
```

# 17、EVENT NOTIFICATION #
事件通知
```
notify-keyspace-events Elg 
```
## 17.1、notify-keyspace-events ##
键空间通知使得客户端可以通过订阅频道或模式，来接收那些以某种方式改动了 Redis 数据集的事件。因为开启键空间通知功能需要消耗一些 CPU ，所以在默认配置下，该功能处于关闭状态。
notify-keyspace-events 的参数可以是以下字符的任意组合，它指定了服务器该发送哪些类型的通知：
- K 键空间通知，所有通知以 \_\_keyspace@\_\_ 为前缀
- E 键事件通知，所有通知以 \_\_keyevent@\_\_ 为前缀
- g DEL、 EXPIRE 、 RENAME 等类型无关的通用命令的通知
- $ 字符串命令的通知
- l 列表命令的通知
- s 集合命令的通知
- h 哈希命令的通知
- z 有序集合命令的通知
- x 过期事件：每当有过期键被删除时发送
- e 驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送
- A 参数 g$lshzxe 的别名

输入的参数中至少要有一个 K 或者 E，否则的话，不管其余的参数是什么，都不会有任何 通知被分发。示例如下 ：
```
notify-keyspace-events Elg
```

# 18、 ADVANCED CONFIG #
高级配置

```
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```

## 18.1、hash-max-ziplist-entries ##
hash类型的数据结构在编码上可以使用ziplist和hashtable。
ziplist的特点就是文件存储(以及内存存储)所需的空间较小,在内容较小时,性能和hashtable几乎一样。
因此redis对hash类型默认采取ziplist。如果hash中条目的条目个数或者value长度达到阀值,将会被重构为hashtable。
这个参数指的是ziplist中允许存储的最大条目个数，，默认为512，建议为128
```
hash-max-ziplist-entries 128
```
## 18.2、hash-max-ziplist-value ##
ziplist中允许条目value值最大字节数，默认为64，建议为1024
```
hash-max-ziplist-value 1024
```
## 18.3、list-max-ziplist-size ##
当取正值的时候，表示按照数据项个数来限定每个quicklist节点上的ziplist长度。比如，当这个参数配置成5的时候，表示每个quicklist节点的ziplist最多包含5个数据项。
当取负值的时候，表示按照占用字节数来限定每个quicklist节点上的ziplist长度。这时，它只能取-1到-5这五个值，每个值含义如下：
- -5 每个quicklist节点上的ziplist大小不能超过64 Kb。（注：1kb => 1024 bytes）
- -4 每个quicklist节点上的ziplist大小不能超过32 Kb。
- -3 每个quicklist节点上的ziplist大小不能超过16 Kb。
- -2 每个quicklist节点上的ziplist大小不能超过8 Kb。（-2是Redis给出的默认值）
- -1 每个quicklist节点上的ziplist大小不能超过4 Kb

## 18.4、list-compress-depth ##
这个参数表示一个quicklist两端不被压缩的节点个数。
注：这里的节点个数是指quicklist双向链表的节点个数，而不是指ziplist里面的数据项个数。
实际上，一个quicklist节点上的ziplist，如果被压缩，就是整体被压缩的。
参数list-compress-depth的取值含义如下：
- 0: 是个特殊值，表示都不压缩。这是Redis的默认值。
- 1: 表示quicklist两端各有1个节点不压缩，中间的节点压缩。
- 2: 表示quicklist两端各有2个节点不压缩，中间的节点压缩。
- 3: 表示quicklist两端各有3个节点不压缩，中间的节点压缩。
  依此类推…

由于0是个特殊值，很容易看出quicklist的头节点和尾节点总是不被压缩的，以便于在表的两端进行快速存取

## 18.4、set-max-intset-entries ##

set允许存储的最大条目个数小于等于set-max-intset-entries用intset，大于set-max-intset-entries用set

## 18.5、zset-max-ziplist-entries ##
zset中允许存储的最大条目个数小于等于zset-max-ziplist-entries用ziplist，大于zset-max-ziplist-entries用zset

## 18.6、zset-max-ziplist-value ##
zset中允许的value值最大字节数小于等于zset-max-ziplist-value用ziplist，大于zset-max-ziplist-value用zset

## 18.7、hll-sparse-max-bytes ##
设置的值小于等于hll-sparse-max-bytes使用稀疏数据结构（sparse）
大于hll-sparse-max-bytes使用稠密的数据结构（dense），取值范围在 0 ~ 15000，
建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右

## 18.8、activerehashing ##
Redis将在每100毫秒时使用1毫秒的CPU时间来对redis的hash表进行重新hash，可以降低内存的使用。
当你的使用场景中，有非常严格的实时性需要，不能够接受Redis时不时的对请求有2毫秒的延迟的话，把这项配置为no。
如果没有这么严格的实时性要求，可以设置为yes，以便能够尽可能快的释放内存

## 18.9、client-output-buffer-limit normal ##
对客户端输出缓冲进行限制可以强迫那些不从服务器读取数据的客户端断开连接，用来强制关闭传输缓慢的客户端。
```
client-output-buffer-limit normal 0 0 0
```
对于normal client，第一个0表示取消hard limit，第二个0和第三个0表示取消soft limit，normal client默认取消限制，因为如果没有寻问，他们是不会接收数据的

## 18.10、client-output-buffer-limit ##
```
client-output-buffer-limit slave 256mb 64mb 60
```
对于slave client和MONITER client，如果client-output-buffer一旦超过256mb，又或者超过64mb持续60秒，那么服务器就会立即断开客户端连接

## 18.11、client-output-buffer-limit ##
```
client-output-buffer-limit pubsub 32mb 8mb 60
```
对于pubsub client，如果client-output-buffer一旦超过32mb，又或者超过8mb持续60秒，那么服务器就会立即断开客户端连接

## 18.12、hz ##

```
hz 10
```
redis执行任务的频率为1s除以hz ，如  1/10


## 18.13、aof-rewrite-incremental-fsync ##
```
aof-rewrite-incremental-fsync yes
```
在aof重写的时候，如果打开了aof-rewrite-incremental-fsync开关，系统会每32MB执行一次fsync。
这对于把文件写入磁盘是有帮助的，可以避免过大的延迟峰值


# 19、ACTIVE DEFRAGMENTATION #
活跃的碎片整理
注意，此功能在 此系列教程使用的版本中只是实验性的，您也可以无视它。然而，即使是在生产过程中，它也经受了压力测试，并由多名工程师手工测试了一段时间。

什么是活跃的碎片整理？
活动(联机)碎片整理允许Redis服务器压缩内存中数据的小分配和deal位置之间的空间，从而恢复内存。

碎片化是一个自然过程，每一个分配器和某些工作负载都会发生。通常需要重新启动服务器以降低碎片，或者至少清除所有数据并重新创建它。然而，由于Oran Agra实现了Redis 4.0的这个特性，这个过程在运行时可以在运行时以“热”的方式运行。

在redis运行的情况下，此功能默认是禁用的，只有在编译Redis时才会开启。
```
activedefrag yes
active-defrag-ignore-bytes 100mb
active-defrag-threshold-lower 10
active-defrag-threshold-upper 100
active-defrag-cycle-min 25
active-defrag-cycle-max 75
```
## 19.1、activedefrag ##
```
activedefrag yes
```
启用碎片整理

## 19.1、active-defrag-ignore-bytes ##
碎片垃圾开始主动整理的最低大小

## 19.2、active-defrag-threshold-lower ##
碎片开始自动整理的最小百分比

## 19.3、active-defrag-threshold-upper ##
最大化碎片率

## 19.4、active-defrag-cycle-min ##
最小整理磁盘碎片的CPU百分比

## 19.5、active-defrag-cycle-max ##
最大整理磁盘碎片的CPU百分比