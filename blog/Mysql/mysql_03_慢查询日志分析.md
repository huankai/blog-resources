---
title: Mysql_03_慢查询日志分析
date: {{ date }}
author: huangkai
tags:
    - Mysql
---

# 一、 使用  mysqldumpslow #

mysql的慢查询日志是MYSQL提供的一种日志记录，它用来记录在MYSQL中响应超时阈值的语句，具体指运行时间超过 `long_query_time` 值的SQL，则会被记录到慢查询日志中去，此参数的默认值为 10 秒 ，可以使用 `SHOW VARIABLES LIKE '%long_query_time%'` 查看。

## 1、查看是否开启了慢查询 ##
如果不是调优需要的话，不般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的影响，只有大于(不包含等于)了此阈值的慢查询日志支持将日志记录到文件或表中。
如下，标识mysql的慢查询日志未开启，默认也为未开启。
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_32.png)

## 2、开启慢查询日志： ##
- 使用命令,只对当前数据库生效，重启数据库会失效
```
set global slow_query_log=1;
```
- 使用配置
修改配置文件在[mysqld]内添加 如下内容， 重启mysql，永久有效。

```
#开启慢查询
slow_query_log=1
#指定慢查询日志路径，默认会保存在 /var/mysql/${hostname}-slow.log 中。
slow_query_log_file=/var/mysql/test-slow.log
```


## 3、修改慢查询日志阈值： ##
设置为3秒，需要重新开启session才能生效，如下
```
set global long_query_time=3;
```

## 4、测试： ##

执行 SQL:
```
SELECT SLEEP(4);
```
查看日志内容,，如下：
```
[huangkai@ mysql]$ sudo cat 8f45c42d2296-slow.log 
/usr/sbin/mysqld, Version: 8.0.13 (MySQL Community Server - GPL). started with:
Tcp port: 3306  Unix socket: /var/run/mysqld/mysqld.sock
Time                 Id Command    Argument
# Time: 2019-01-14T09:07:17.195193Z
# User@Host: root[root] @  [192.168.64.128]  Id:  1396
# Query_time: 4.000558  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1547456837;
SELECT SLEEP(4);
```

## 5、查看慢查询日志记录数： ##
如下，如果 value 值越大，慢sql就会越多。
```
SHOW GLOBAL STATUS LIKE '%slow_queries%';
```

## 6、日志分析工具 mysqldumpslow ##
mysqldumpslow 是 mysql 提供的慢sql日志分析工具

使用帮助如下：
```
root@8f45c42d2296:/# mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default （表示按何种方式排序）
                al: average lock time（平均锁定时间）
                ar: average rows sent (平均返回记录数)
                at: average query time (平均查询时间)
                 c: count（访问次数）
                 l: lock time(锁定时间)
                 r: rows sent(返回记录)
                 t: query time  (查询时间)
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries（返回当前多少条记录）
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names（后面搭配正则表达式，大小写不敏感）
  -g PATTERN   grep: only consider stmts that include this string
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time

root@8f45c42d2296:/# 
```
如: 查询返回记录最多的 10 个SQL
如下，表示 SELECT SLEEP(N) 这条SQL
```
root@8f45c42d2296:/# mysqldumpslow -s r -t 10 /var/lib/mysql/8f45c42d2296-slow.log 

Reading mysql slow query log from /var/lib/mysql/8f45c42d2296-slow.log
Count: 2  Time=4.50s (9s)  Lock=0.00s (0s)  Rows=1.0 (2), root[root]@[192.168.64.128]
  SELECT SLEEP(N)

Died at /usr/bin/mysqldumpslow line 166, <> chunk 2.
```

如: 查询返回次最多的 10 个SQL
如下，表示 SELECT SLEEP(N) 这条SQL
```
root@8f45c42d2296:/# mysqldumpslow -s c -t 10 /var/lib/mysql/8f45c42d2296-slow.log  

Reading mysql slow query log from /var/lib/mysql/8f45c42d2296-slow.log
Count: 2  Time=4.50s (9s)  Lock=0.00s (0s)  Rows=1.0 (2), root[root]@[192.168.64.128]
  SELECT SLEEP(N)

Died at /usr/bin/mysqldumpslow line 166, <> chunk 2.
root@8f45c42d2296:/#
```

# 二、使用 Show Profile #
使用 Show Profile 相比 mysqldumpslow 能更加细粒度的分析SQL，可以分析 sql语句的整个执行资源消耗情况，查看SQL执行的生命周期，可以用于SQL的调优测量。默认情况下，该参数处于关闭状态，并保存最近 15 次的运行结果，可以使用 `SHOW VARIABLES LIKE 'profiling'` 查看是否开启，如下：。
```
mysql> SHOW VARIABLES LIKE 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+
1 row in set (0.02 sec)

mysql> 

####################### 如果是关闭状态，可以使用 `set profiling=on;` 开启。
mysql> set profiling=on;
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW VARIABLES LIKE 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | ON    |
+---------------+-------+
1 row in set (0.02 sec)

mysql> 

######################  诊断SQL:
mysql> SHOW PROFILES;
+----------+------------+---------------------------------+
| Query_ID | Duration   | Query                           |
+----------+------------+---------------------------------+
|        1 | 0.00257725 | SHOW VARIABLES LIKE 'profiling' |
+----------+------------+---------------------------------+
1 row in set (0.02 sec)

mysql> insert into article(title,category_id,comments,views) select title,category_id,comments,views from article;
Query OK, 1 row affected (0.11 sec)
Records: 1  Duplicates: 0  Warnings: 0
mysql> 

#######################  这里多次执行这条sql，将  article 表中插入 50多万条记录
mysql> insert into article(title,category_id,comments,views) select title,category_id,comments,views from article;
Query OK, 2 rows affected (0.08 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select count(*) from article;
+----------+
| count(*) |
+----------+
|   524288 |
+----------+
1 row in set (0.11 sec)
mysql> 

####################### 使用 SHOW PROFILES;会得到最近操作的SQL语句 
mysql> SHOW PROFILES;
+----------+-------------+------------------------------------------------------------------------------------------------------------+
| Query_ID | Duration    | Query                                                                                                      |
+----------+-------------+------------------------------------------------------------------------------------------------------------+
|        9 |  0.06134625 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       10 |  0.10083725 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       11 |  0.09136375 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       12 |  0.07717400 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       13 |  0.07325425 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       14 |  0.05692050 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       15 |  0.10081450 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       16 |  0.11223675 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       17 |  0.18224725 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       18 |  0.31501000 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       19 |  0.78707675 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       20 |  1.04923400 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       21 |  2.12224800 | insert into article(title,category_id,comments,views) select title,category_id,comments,views from article |
|       22 |  0.07427625 | select count(*) from article                                                                               |
|       23 | 30.71403600 | select * from article limit 150000                                                                         |
+----------+-------------+------------------------------------------------------------------------------------------------------------+
15 rows in set (0.03 sec)
mysql> 

####################### SHOW PROFILE cpu,block io FOR QUERY query_id;-- query_id 是指上面 SHOW PROFILES 列表中的 Query_ID值，如下，表示执行第23条Query_ID的sql在 Sending data 占用的时间比较长。
mysql> SHOW PROFILE cpu,block io FOR QUERY 23;
+----------------------------+------------+----------+------------+--------------+---------------+
| Status                     | Duration   | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------------+------------+----------+------------+--------------+---------------+
| starting                   | 0.000188   | 0.000176 | 0.000000   |            0 |             0 |
| checking permissions       | 0.000018   | 0.000016 | 0.000000   |            0 |             0 |
| Opening tables             | 0.000077   | 0.000076 | 0.000000   |            0 |             0 |
| init                       | 0.000013   | 0.000012 | 0.000000   |            0 |             0 |
| System lock                | 0.000018   | 0.000018 | 0.000000   |            0 |             0 |
| optimizing                 | 0.000010   | 0.000009 | 0.000000   |            0 |             0 |
| statistics                 | 0.000027   | 0.000028 | 0.000000   |            0 |             0 |
| preparing                  | 0.000020   | 0.000020 | 0.000000   |            0 |             0 |
| executing                  | 0.000007   | 0.000007 | 0.000000   |            0 |             0 |
| Sending data               | 63.540117  | 1.122394 | 0.314036   |            0 |             0 |
| end                        | 0.000044   | 0.000000 | 0.000033   |            0 |             0 |
| query end                  | 0.000017   | 0.000000 | 0.000017   |            0 |             0 |
| waiting for handler commit | 0.000021   | 0.000000 | 0.000019   |            0 |             0 |
| query end                  | 0.000017   | 0.000000 | 0.000016   |            0 |             0 |
| closing tables             | 0.000027   | 0.000000 | 0.000026   |            0 |             0 |
| freeing items              | 0.000021   | 0.000000 | 0.000021   |            0 |             0 |
| cleaning up                | 0.000038   | 0.000000 | 0.000038   |            0 |             0 |
+----------------------------+------------+----------+------------+--------------+---------------+
17 rows in set (0.06 sec)

mysql> 

```
以上只显示了 cpu和 block io 信息，如果你想查看其它信息，可以使用 
- ALL 显示所有信息
- BLOCK IO  显示块IO相关开销
- CONTEXT SWITCHES	上下文切换相关开销
- CPU	显示CPU 相关开销信息	
- IPC	显示发送和接收相关开销信息
- MEMORY	显示内存相关开销信息
- PAGE FAULTS	显示页面错误相关开销信息
- SOURCE		显示和Source_function,Source_file,Source_line 相关的开销信息
- SWAPS		显示交换次数相关开销的信息

如 ：

```
mysql>  SHOW PROFILE cpu,block io FOR QUERY 23;
.......
```
在上 Status　列中，如果出现以下列之一，必须优化。

- converting HEAP to MyISAM 查询结果太大，内存不够用了，往磁盘上搬了
- Creating tmp table 创建临时表拷贝数据,用完再删除。
- Copying to tmp table on disk 把内存中临时表复制到磁盘，危险！！！
- locked


# 三、全局查询日志 #
<b style='color:red;'>永远不要在生产环境开启这个功能。</b>

## 3.1、查看是否开启全局日志 ##
```
mysql> SHOW VARIABLES LIKE 'general_log';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| general_log   | OFF   |
+---------------+-------+
1 row in set (0.02 sec)

mysql> 
```

## 3.2、开启全局日志 ##

- 配置文件启用：
在my.conf文件中 添加如下内容，重启MYSQL。
```
general_log=1
#记录日志文件到指定目录
general_log_file=/path/logfile
#输出格式：文件 或表，如果设置为 TABLE，会在 mysql 库中创建 general_log 表。
log_output=FILE/TABLE
```
- 命令启用：
```
set global general_log=1;
set global log_output='TABLE'
```

## 3.3、检查结果 ##
开启后，你所编写的SQL语句，将会记录到Mysql库里的 general_log 表中，可以使用如下命令查看：
```
sql> select * from mysql.general_log;
```