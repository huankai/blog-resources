---
title: Mysql_05_ 主从配置
date: {{ date }}
author: huangkai
tags:
    - Mysql
---

# 一、环境配置 #
|hostname| IP |说明|
|:-:|:-:|:-:|
|sjq128|192.168.164.128|主|
|sjq129|192.168.164.129|从|

安装版本: 8.0.13
操作系统版本： CentOS Linux release 7.3.1611 (Core) 

<b style="color:red;">说明：在配置主从前，先将每台服务器的数据库与表创建好，保证两台服务器中的数据先同步。</b>
# 二、安装(使用Docker) #
两台服务器同时执行：
```
[huangkai@sjq129 ~]$ cat /etc/redhat-release 
CentOS Linux release 7.3.1611 (Core) 
[huangkai@sjq128 ~]$ docker-compose -v
docker-compose version 1.23.2, build 1110ad01
[huangkai@sjq128 ~]$ docker version
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:23:03 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:25:29 2018
  OS/Arch:          linux/amd64
  Experimental:     false
[huangkai@sjq128 ~]$ cd /data/docker/
[huangkai@sjq128 docker]$ ll
total 4
-rw-rw-r--. 1 huangkai huangkai 808 Jan 11 17:34 docker-compose.yml
[huangkai@sjq128 ~]$ cd /data/docker/
[huangkai@sjq128 docker]$ mkdir mysql8/conf -p
[huangkai@sjq128 docker]$ mkdir mysql8/data -p
[huangkai@sjq128 docker]$ vim docker-compose.yml
############################### 添加如下配置
services:
  mysql8:
    image: mysql:8.0.13
    container_name: mysql8
    ports:
      - "3306:3306"
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/data/docker/mysql8/data/:/var/lib/mysql"
      - "/data/docker/mysql8/conf/:/etc/mysql/conf.d"
    restart: "always"
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: 'true'
############################### 
[huangkai@sjq128 docker]$ vim mysql8/conf/mysql.cnf
############################### 添加如下内容

[mysqld]
#忽略表名大小写
lower_case_table_names=1


character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect=’SET NAMES utf8mb4'

[mysql]

default-character-set=utf8mb4

[client]
default-character-set=utf8
############################### 

[huangkai@sjq128 docker]$ docker-compose up -d mysql8
```
# 三、主配置 #

修改 `/data/docker/mysql8/conf/mysql.cnf` 文件，内容如下：

```
[mysqld]
#忽略表名大小写
lower_case_table_names=1

# server id，全局唯一
server-id=128
# 忽略需要同步的数据库名，可以指定多个
binlog-ignore-db=mysql

# 需要同步的数据库名，可以指定多个
binlog-do-db=db1
binlog-do-db=db2

# 开启二进制日志
log-bin=sjq-mysql-bin

# 配置 binlog 缓存大小
binlog_cache_size=2M

# 配置 binlog 格式
binlog_format=mixed

# 二进制日志自动删除的时间，默认值为0,表示“没有自动删除”，启动时和二进制日志循环时可能删除
# 在 mysql 8 以前 ，参数为 expire_logs_days ，
# 在 mysql 8 以后，此参数已过时，用 binlog_expire_logs_seconds 代替，单位为秒
#expire_logs_days=7
binlog_expire_logs_seconds=2592000

# 将函数复制到slave,默认为 false. 
log_bin_trust_function_creators = 1

# mysql在主从复制过程中，由于各种的原因，从服务器可能会遇到执行BINLOG中的SQL出错的情况，在默认情况下，服务器会停止复制进程，不再进行同步，等到用户自行来处理
# slave-skip-errors的作用就是用来定义复制过程中从服务器可以自动跳过的错误号，当复制过程中遇到定义的错误号，就可以自动跳过，直接执行后面的SQL语句
# slave_skip_errors选项有四个可用值，分别为：off，all，ErorCode，ddl_exist_errors。
# 默认情况下该参数值是off，我们可以列出具体的error code，也可以选择all，mysql5.6及MySQL Cluster NDB 7.3以及后续版本增加了参数ddl_exist_errors，该参数包含一系列error code（1007,1008,1050,1051,1054,1060,1061,1068,1094,1146）
#  一些error code代表的错误如下：
#    1007：数据库已存在，创建数据库失败
#    1008：数据库不存在，删除数据库失败
#    1050：数据表已存在，创建数据表失败
#    1051：数据表不存在，删除数据表失败
#    1054：字段不存在，或程序文件跟数据库有冲突
#    1060：字段重复，导致无法插入
#    1061：重复键名
#    1068：定义了多个主键
#    1094：位置线程ID
#    1146：数据表缺失，请恢复数据库
#    1053：复制过程中主服务器宕机
#    1062：主键冲突 Duplicate entry '%s' for key %d
slave_skip_errors=1062,1053
# slave_skip_errors=all
# slave_skip_errors=ddl_exist_errors

[client]
default-character-set=utf8
```
更多参数配置可以查看： https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html

重启 mysql:
```
[huangkai@sjq128 docker]$ docker-compose restart mysql8
```

登陆MySQL：执行如下：
```
mysql> CREATE USER 'repl'@'%' IDENTIFIED BY 'root';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
mysql> flush privileges;
mysql> show master status;
+----------------------+----------+--------------+------------------+-------------------+
| File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------------+----------+--------------+------------------+-------------------+
| sjq-mysql-bin.000001 |      867 | db1,db2      | mysql            |                   |
+----------------------+----------+--------------+------------------+-------------------+
1 row in set (0.02 sec)

mysql>
```

# 四、从配置 #

mysql.cnf 内容如下：
```
[mysqld]
#忽略表名大小写
lower_case_table_names=1
# server id ，全局唯一
server-id=129
# 忽略同步的数据库名
binlog-ignore-db=mysql 
# bin log
log-bin=sjq-mysql-slave1-bin 
#
binlog_cache_size = 2M
#
binlog_format=mixed 
#
# expire_logs_days=7 
binlog_expire_logs_seconds=2592000
#
slave_skip_errors=1062,1053
#
relay_log=sjq-mysql-relay-bin
#
log_slave_updates=1
#
read_only=1
[client]
default-character-set=utf8
```

重启 mysql，登陆MySQL：执行如下：
```
mysql> CHANGE MASTER TO MASTER_HOST='192.168.64.128',MASTER_USER='repl',MASTER_PASSWORD='root',MASTER_LOG_FILE='sjq-mysql-bin.000001',MASTER_LOG_POS=867;
Query OK, 0 rows affected (0.14 sec)

################ 开启主从
mysql> start slave; 

```


检查如下两列的值是否都为 Yes ，如果是，主从配置成功。
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_40.png)

停止主从：
在 从服务器上执行：
```
mysql> stop slave; 
```
# 五、测试 #

在主服务器中某表中创建或修改一条记录，查看从服务器相同库表的数据是否有更新。