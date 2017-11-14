---
title: Redis 持久化
date: {{ date }}
author: huangkai
tags: 
	- Redis
---

Redis提供了不同级别的持久化方式:
- RDB(能够在指定的时间间隔对数据进行快照存储)
- AOF(记录每次对服务器进行写操作，当服务器重启时会执行这此命令恢复原始数据，AOF以redis协议追加保存每次写操作到文件末尾，Redis还支持对AOF文件重写，使得AOF文件体积不会过大)
- 如果只希望数据在服务器运行时存在，你可以不使用任何持久化方式
- 也可以同时开启两种方式(RDB和AOF),这种情况下，当Redis服务重启时，会优先加载AOF来恢复数据，因为AOF文件保存要比RDB保存的数据集更完整。


# 1、RDB: #

RDB持久化方式能够在指定的时间间隔能对数据进行快照存储。
## RDB的优点： ##
- RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.
- RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心，非常适用于灾难恢复.
- RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.
- 与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.

## RDB的缺点： ##
- 虽然你可以配置save的时间点(如 5分钟内有1000个写操作，执行一次持久化)，但redis要完整保存数据集是一个比较繁重的工作,当Redis意外停止工作时，可能会丢失几分钟之内的数据
- RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度。

RDB默认是开启的，配置内容如下，如果不需要开始，注释所有的save 即可。
```
#   save ""
save 900 1
save 300 10
save 60 10000
rdbcompression yes
stop-writes-on-bgsave-error yes
rdbchecksum yes
dbfilename "dump.rdb"
dir "/usr/local/redis-4.0.1"
```

# 2、AOF #

AOF默认是不开启的，aof所有配置项如下，
```
appendonly no
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb，默认为 64mb,建议至少设置 3gb以上。
aof-load-truncated yes
aof-use-rdb-preamble no
```
aof文件损坏了怎么办?

服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：
- 为现有的 AOF 文件创建一个备份。
- 使用 Redis 附带的 redis-check-aof 程序，对原来的 AOF 文件进行修复:
  $ redis-check-aof –fix appendonly.aof
- （可选）使用 diff -u 对比修复后的 AOF 文件和原始 AOF 文件的备份，查看两个文件之间的不同之处。
- 重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复