---
title: Hadoop - HDFS工作机制
date: {{ date }}
author: huangkai
tags:
    - Hadoop
---

NameNode负责管理整个文件系统元数据，DataNode负责具体的文件数据块存储，Secondary NameNode 主要协助NameNode进行元数据的备份。

HDFS的内部工作机制对客户端都是透明的，客户端请求访问HDFS 都需要通过NameNode申请来访问。


# 一、HDFS命令操作： #

## 1.1、显示目录信息(ls) ##

```
[huangkai@sjq-01 hadoop-2.9.1]$ hadoop fs -ls /
```

## 1.2、在hdfs上创建目录(mkdir) ##

-p 参数指定级联创建目录
```
[huangkai@sjq-01 hadoop-2.9.1]$ hadoop fs -mkdir -p /aaa/bbb/cc/dd
```

## 1.3、本地剪切粘贴到hdfs(moveFromLocal、copyFromLocal、put) ##

假设在 `/home/huangkai` 目录下存在文件 `a.txt` 

```
[huangkai@sjq-01 hadoop-2.9.1]$ hadoop fs -moveFromLocal /home/huangkai/a.txt  /aaa/bbb/cc/dd
```

也可以使用 copyFromLocal

```
[huangkai@sjq-01 hadoop-2.9.1]$ hadoop fs -copyFromLocal /home/huangkai/a.txt  /aaa/bbb/cc/dd
```

也可以使用 put

```
[huangkai@sjq-01 hadoop-2.9.1]$ hadoop fs -put /home/huangkai/a.txt  /aaa/bbb/cc/dd
```


## 1.4、追加一个文件到已存在的文件末尾(appendToFile) ##

假设在 `/home/huangkai` 目录下存在文件 `b.txt`，文件内容为 `aaaaabbbbbbbbbbbb`

```
[huangkai@sjq-01 ~]$ hadoop fs -appendToFile  ./b.txt  /aaa/bbb/cc/dd/a.txt
```

## 1.5、查看文件内容(cat) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -cat /aaa/bbb/cc/dd/a.txt
abcd
defa
aaaaabbbbbbbbbbbb
[huangkai@sjq-01 ~]$
```

## 1.6、 显示文件的末尾内容(tail) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -tail /aaa/bbb/cc/dd/a.txt
abcd
defa
aaaaabbbbbbbbbbbb
[huangkai@sjq-01 ~]$
```

## 1.7、以字符串形式显示文件内容(text) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -text /aaa/bbb/cc/dd/a.txt
abcd
defa
aaaaabbbbbbbbbbbb
[huangkai@sjq-01 ~]$
```

## 1.8、修改用户所属权限(chmod、chown、chgrp) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -chmod 666 /aaa/bbb/cc/dd/a.txt
[huangkai@sjq-01 ~]$ hadoop fs -chown huangkai:huangkai /aaa/bbb/cc/dd/a.txt
```

## 1.9、从hdfs的一个路径拷贝到hdfs的另一个路径(cp) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -cp /aaa/bbb/cc/dd/a.txt  /aaa/bbb/jdk.tar.gz
```

## 1.10、从hdfs的一个路径移动到hdfs的另一个路径(mv) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -mv /aaa/bbb/cc/dd/a.txt  /aaa
```

## 1.11、从hdfs 下载(get、copyToLocal) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -get /aaa/a.txt
```

还可以使用 

```
[huangkai@sjq-01 ~]$ hadoop fs -copyToLocal /aaa/a.txt
```

## 1.12、从hdfs 下载多个文件并合并(getmerge) ##

假设 hdfs 上的 `/aaa`目录下有多个`log`开头的文件，将其下载到当前目录 `log.sum` 文件，
```
[huangkai@sjq-01 ~]$ hadoop fs -getmerge /aaa/log.* ./log.sum
```

## 1.13、删除文件或文件夹(rm) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -rm -r /aaa/bbb/cc/dd
Deleted /aaa/bbb/cc/dd
[huangkai@sjq-01 ~]$ 
```
## 1.13、删除空目录(rmdir) ##

如果目录不为空，不能删除
```
[huangkai@sjq-01 ~]$ hadoop fs -rmdir /aaa/bbb/cc
```

## 1.14、统计文件系统的可用空间信息(df) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -df -h /
Filesystem                   Size     Used  Available  Use%
hdfs://hadoop-node-1:9000  40.9 G  110.7 K     33.3 G    0%
[huangkai@sjq-01 ~]$ 
```

## 1.15、统计文件夹的大小信息(du) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -du -s -h /aaa
56  /aaa
[huangkai@sjq-01 ~]$ 
```

## 1.16、统计一个指定目录下的文件节点数量(count) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -count /aaa
           2            2                 56 /aaa
[huangkai@sjq-01 ~]$ 
```

## 1.17、设置hdfs文件副本数(setrep) ##

```
[huangkai@sjq-01 ~]$ hadoop fs -setrep 3 /aaa/bbb/jdk.tar.gz
Replication 3 set: /aaa/bbb/jdk.tar.gz
[huangkai@sjq-01 ~]$ 
```

更多命令请查看 [官网文档](https://hadoop.apache.org/docs/r2.9.1/hadoop-project-dist/hadoop-common/FileSystemShell.html)

# 二、JAVA API 操作HDFS #

请查看:

https://github.com/huankai/hk-hadoop/tree/master/hk-hadoop-helloworld