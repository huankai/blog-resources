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