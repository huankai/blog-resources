---
title: Mysql_06_备份
date: {{ date }}
author: huangkai
tags:
    - Mysql
---

一、使用 mysqldump 备份
===============

1、docker 容器中的mysql备份
------------------


```
#!/bin/bash

#指定容器 id
container_name=c128cdfef408

# 指定备份目录,如果不存在，则创建

data_dir=/root/back/mysql8/xls_fs
if [ ! -d "$data_dir" ]; then
  mkdir -p $data_dir
fi

# 执行 docker 备份，注意，这里不需要添加 -it 参数,如果添加了在使用 crontab 定时执行的时候dump出来的文件大小始终是0，去掉-it就可以了，按照文档解释-t是分配一个伪终端,但是crontab执行的时候实际是不需要的
/usr/bin/docker exec  ${container_name} mysqldump -uroot -proot --databases xls_fs > "$data_dir/data_`date +%Y%m%d%H%M%S`.sql"

#删除 7 天前的备份
find $data_dir -mtime +7 -name 'data_[1-9].sql' -exec rm -rf {} \;
```

crontab -e 添加如下内容，每天早上 1点备份：

```
[root@localhost mysql8]# crontab -e
* 1 * * * /root/back/mysql8/xls_fs.sh
```

查看定时任务：
```
[root@localhost mysql8]# crontab -l
* 1 * * * /root/back/mysql8/xls_fs.sh
```

二、使用 XtraBackup 备份
==================
2.1、介绍与安装
------
https://www.percona.com/doc/percona-xtrabackup/8.0/intro.html

XtraBackup 功能：
- 创建热门InnoDB备份而不暂停数据库
- 进行MySQL的增量备份
- 将压缩的MySQL备份流式传输到另一台服务器
- 在线运行MySQL服务器之间的表
- 轻松创建新的MySQL复制从属服务器
- 备份MySQL而不向服务器添加负载

下载地址：
https://www.percona.com/downloads/Percona-XtraBackup-LATEST/


安装：
```
[huangkai@instance-3knenqmy ~]$ wget https://www.percona.com/downloads/Percona-XtraBackup-LATEST/Percona-XtraBackup-8.0-7/binary/redhat/7/x86_64/percona-xtrabackup-80-8.0.7-1.el7.x86_64.rpm

[huangkai@instance-3knenqmy ~]$ yum localinstall percona-xtrabackup-80-8.0.7-1.el7.x86_64.rpm
```

2.2、 安装 mysql 并配置备份用户
-----------------

docker 安装mysql，docker-compose 配置内容如下：
```
version: "3.7"
services:
  mysql8: 
    image: 172.16.0.4:8870/mysql:8.0.17
    container_name: mysql8
    ports:
      - "3306:3306"
    volumes: 
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/data/docker/mysql8.0.17/data/:/var/lib/mysql"
      - "/data/docker/mysql8.0.17/conf/:/etc/mysql/conf.d"
    restart: "always"
    environment: 
      #如果不指定密码，用户只能本机登陆，且root密码为空
      MYSQL_ALLOW_EMPTY_PASSWORD: 'true'
      #MYSQL_ROOT_PASSWORD: 'root'

```
启动 mysql ：

```
[huangkai@localhost docker]$ docker-compose up -d mysql8
```
配置备份用户：

```

mysql> CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY 's3cr%T';
mysql> GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost';
mysql> GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
mysql> FLUSH PRIVILEGES;
```

创建软链接(可选)

上面在docker 宿主机中安装了 xtraBackup 工具，但是这个功能在备份时需要指定 mysql 的datadir 或 mysql的配置文件，上面docker 中挂载的 mysql 的数据存储目录与宿主机中的不一样，docker容器中使用的 `/var/lib/mysql` 目录，而宿主机中使用的是 `/data/docker/mysql8.0.16/data`目录， 而 xtraBackup 在备份时会读取 mysql 的 data 目录，这里指定的是 docker 容器中的目录，即`/var/lib/mysql` ，有两种方式
- docker 在挂载时的 mysql 存储目录与宿主机的一样，并在 mysql.conf 配置文件中指定 datadir所在的目录。
- 如果不一样，可以将宿主机中使用软链接将docker容器中使用的目录(`/var/lib/mysql`)与 xtraBackup 读取宿主机的目录相关联。如<font color='red'>`ln -s /data/docker/mysql8.0.16/data /var/lib/mysql`</font> ，注意 `/data/docker/mysql8.0.16/data` 不能写成 `/data/docker/mysql8.0.16/data/`,最后加了 `/` 会导致创建的软连接多一层目录


## 2.3、全量备份脚本 ##


```
[root@instance-3knenqmy mysql8]# vim /root/back/mysql8/full_back.sh

#!/bin/bash

# mysql  数据存放目录
MYSQL_DATA_DIR=/var/lib/mysql

# mysql 端口号
MYSQL_PORT=3707
# MYSQL 主机
MYSQL_HOST=172.17.0.1
#mysql 用户名
MYSQL_USER=root
# mysql 密码
MYSQL_PASSWORD=root
MY_CNF=/data/docker/mysql8.0.17/conf/mysql.cnf  #mysql的配置文件
#mysql 的连接信息
MYSQL_CMD="--host=$MYSQL_HOST --port=$MYSQL_PORT  --user=$MYSQL_USER --password=$MYSQL_PASSWORD"

#  全量备份目录
FULL_BACKUP_BASE_DIR=/root/back/mysql8/full

#全量备份目录 ，以时间分隔
FULL_DATA_DIR=$FULL_BACKUP_BASE_DIR/`date +%Y%m%d%H%M%S`

XTRABACKUP_FULL=/usr/bin/xtrabackup

# 定义错误方法
error()
{
  echo "$1" 1>&2
  exit 1
}

if [ ! -x $XTRABACKUP_FULL ]; then
  error "$XTRABACKUP_FULL 命令找不到"
fi

echo "$0: MySQL备份脚本，备份时间: `date +%F' '%T' '%w`"

# 创建全量备份目录
mkdir -p $FULL_DATA_DIR
# 执行备份
$XTRABACKUP_FULL --defaults-file=$MY_CNF --backup --use-memory=4G $MYSQL_CMD --target-dir=$FULL_DATA_DIR

#  -----------------------------------删除之前的备份-----------------------------------------------------
#　查询所有全量备份
back_list=(`find $FULL_BACKUP_BASE_DIR -mindepth 1 -maxdepth 1 -type d -printf "%P\n" | sort -n`)

#全量备份保存的个数
KEEP_FULL_BACKUP=2

size=`expr ${#back_list[*]} - $KEEP_FULL_BACKUP`

if [ $size -gt 0 ];then
  cd $FULL_BACKUP_BASE_DIR
  for((i=0;i<$size;i++)); do
    echo "delete backup :" ${back_list[i]}
    rm -rf ${back_list[i]}
  done
fi
```

将上面的增量备份添加到 `crontab`中，并以每5分钟执行一次

```
[root@instance-3knenqmy mysql8]# crontab -l
*/5 * * * * /root/back/mysql8/full_back.sh >/dev/null 2>&1
[root@instance-3knenqmy mysql8]#
```

## 2.4、增量备份脚本 ##

```
[root@instance-3knenqmy mysql8]# vim /root/back/mysql8/incr_back.sh
#!/bin/bash

# mysql  数据存放目录
MYSQL_DATA_DIR=/var/lib/mysql
# mysql 端口号
MYSQL_PORT=3707
# MYSQL 主机
MYSQL_HOST=172.17.0.1
#mysql 用户名
MYSQL_USER=root
# mysql 密码
MYSQL_PASSWORD=root
MY_CNF=/data/docker/mysql8.0.17/conf/mysql.cnf  #mysql的配置文件
#mysql 的连接信息
MYSQL_CMD="--host=$MYSQL_HOST --port=$MYSQL_PORT  --user=$MYSQL_USER --password=$MYSQL_PASSWORD"

# 全量备份目录
FULL_BACKUP_BASE_DIR=/root/back/mysql8/full

# 增量备份目录
INCR_BACKUP_BASE_DIR=/root/back/mysql8/incr
INCR_DATA_DIR=$INCR_BACKUP_BASE_DIR/`date +%Y%m%d%H%M%S`

# 查询最新的增量备份
LATEST_FULL_BACKUP=`find $INCR_BACKUP_BASE_DIR -mindepth 1 -maxdepth 1 -type d -printf "%P\n" | sort -nr | head -1`
incremental_basedir=''
# 如果上次使用的增量备份存在
if [ -n "$LATEST_FULL_BACKUP" ]
then
  incremental_basedir=$INCR_BACKUP_BASE_DIR/$LATEST_FULL_BACKUP
  echo "查询使用上次增量备份: $LATEST_FULL_BACKUP"
else
   LATEST_FULL_BACKUP=`find $FULL_BACKUP_BASE_DIR -mindepth 1 -maxdepth 1 -type d -printf "%P\n" | sort -nr | head -1`
  incremental_basedir=$FULL_BACKUP_BASE_DIR/$LATEST_FULL_BACKUP
  echo "查询全量备份做为  incremental-basedir: $incremental_basedir"
fi

# 创建增量备份目录
mkdir -p $FULL_BACKUP_BASE_DIR
XTRABACKUP_FULL=/usr/bin/xtrabackup
$XTRABACKUP_FULL --backup --target-dir=$INCR_DATA_DIR $MYSQL_CMD --incremental-basedir=$incremental_basedir

# -----------------------删除之前的增量备份 ---------------------------------------
#　查询所有增量备份
back_list=(`find $INCR_BACKUP_BASE_DIR -mindepth 1 -maxdepth 1 -type d -printf "%P\n" | sort -n`)
#增量备份保存的个数
KEEP_FULL_BACKUP=2
size=`expr ${#back_list[*]} - $KEEP_FULL_BACKUP`
if [ $size -gt 0 ];then
  cd $INCR_BACKUP_BASE_DIR
  for((i=0;i<$size;i++)); do
    echo "delete backup :" ${back_list[i]}
    rm -rf ${back_list[i]}
  done
fi
```

## 2.5、数据恢复 ##

查看备份目录结构
```
[root@instance-3knenqmy mysql8]# tree -L 2
.
├── full
│   └── 20190916155615
├── full_back.sh
├── incr
│   └── 20190916155653
├── incr_back.sh

```

执行恢复：
```
[root@instance-3knenqmy mysql8]# xtrabackup --prepare --apply-log-only --target-dir=/root/back/mysql8/full/20190916155615

[root@instance-3knenqmy mysql8]# xtrabackup --prepare --apply-log-only --target-dir=/root/back/mysql8/full/20190916155615 --incremental-dir=/root/back/mysql8/incr/20190916155653

[root@instance-3knenqmy mysql8]# cd /data/docker
[root@instance-3knenqmy docker]# docker-compose stop mysql8 #停止　mysql

[root@instance-3knenqmy docker]# rm -rf mysql8.0.17/data/* #删除mysql data 目录下所有文件
[root@instance-3knenqmy docker]# ll mysql8.0.17/data/
[root@instance-3knenqmy docker]# cd -
/root/back/mysql8
[root@instance-3knenqmy mysql8]# xtrabackup --copy-back --target-dir=/root/back/mysql8/full/20190916155615
[root@instance-3knenqmy docker]# cd -
[root@instance-3knenqmy docker]# ll mysql8.0.17/data/ #　可以查看 mysql 数据已恢复
[root@instance-3knenqmy docker]# chown polkitd:input mysql8.0.17/data/ -R # 授权
[root@instance-3knenqmy docker]# docker-compose restart mysql8 #重启 mysql ，数据已恢复
```
## 2.6、数据恢复思考 ##

一般在企业中，全量备份可以每周一次，增量备份每天一次，比如每天凌晨2点进行增量备份，而恰巧第二天中午12点数据库崩溃了，则从凌晨2点到第二天中午12点的这段时间内是没有进行数据备份的，怎么才能将这些时间内的数据也恢复呢？？？
此时，可以使用 mysql 中的 `binlog` 进行恢复，
查看文档: https://www.cnblogs.com/jian0110/p/9404773.html
