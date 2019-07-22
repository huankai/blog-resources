---
title: Zookeeper_03 事务日志和snapshot清理方式
date: {{ date }}
author: huangkai
tags:
    - Zookeeper
---



Zookeeper运行过程会产生大量的事务日志和snapshot镜像文件，文件的目录是通过zoo.conf的datadir参数指定的，下面我们就说一下如何清理事务日志和snapshot。

清理的方式有如下三种：

一、zookeeper配置自动清理
=================

zookeeper在3.4.0版本以后提供了自动清理snapshot和事务日志的功能通过配置 autopurge.snapRetainCount 和 autopurge.purgeInterval 这两个参数能够实现定时清理了。这两个参数都是在zoo.cfg中配置的：

我们使用的zk版本是：3.4.6，因此可以使用自带的清理功能

autopurge.purgeInterval 这个参数指定了清理频率，单位是小时，需要填写一个1或更大的整数，默认是0，表示不开启自己清理功能。

autopurge.snapRetainCount 这个参数和上面的参数搭配使用，这个参数指定了需要保留的文件数目。默认是保留3个。

示例：
在 ${ZOOKEEPER_HOME}/conf/zoo.cfg 添加如下内容
```
# 保留60个文件
autopurge.snapRetainCount=60
# 保留48小时内的日志
autopurge.purgeInterval=48
```

二、使用自定义清理脚本
===========

clean_zook_log.sh脚本内容如下

```
#!/bin/bash
            
#snapshot file dir
dataDir=/opt/zookeeper/data/version-2
#tran log dir
dataLogDir=/opt/zookeeper/datalog/version-2
logDir=/opt/zookeeper/logs
#Leave 60 files
count=60
count=$[$count+1]
ls -t $dataLogDir/log.* | tail -n +$count | xargs rm -f
ls -t $dataDir/snapshot.* | tail -n +$count | xargs rm -f
ls -t $logDir/zookeeper.log.* | tail -n +$count | xargs rm -f
```
这个脚本保留最新的60个文件，可以将他写到 将这个脚本添加到crontab中，设置为每天凌晨2点？或者其他时间执行即可

```
crontab -e 2 2 * * * /bin/bash /usr/local/zookeeper/bin/clean_zook_log.sh > /dev/null 2>&1

```

三、使用zkCleanup.sh清理
------------------

这个脚本是使用的zookeeper.jar里的org.apache.zookeeper.server.PurgeTxnLog这个class的main函数清理的，因此需要启动一个java进程，比shell清理要重一些。

org.apache.zookeeper.server.PurgeTxnLog文档

```
sh /usr/local/zookeeper/bin/zkCleanup.sh 数据目录 -n 20
```
参数说明
数据目录： /var/zookeeper 20: 保留快照日志的数量

ps.因为zookeeper从3.4.0版本之后提供了对历史事务日志和快照文件的自动清理，所以这个脚本很少使用，另外在生产环境中我们一般采取自动脚本来定点定量清除指定日期的日志文件


参考: https://ningyu1.github.io/site/post/89-zookeeper-cleanlog/
