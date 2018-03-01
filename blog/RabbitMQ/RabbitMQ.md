---
title: RabbitMQ 安装
date: {{ date }}
author: huangkai
tags:
    - RabbitMQ
---
1 、安装erlang :
安装erlang需要依赖的库：
```
[root@huangkai data]# yum -y install gcc gcc-c++  m4 ncurses-devel openssl-devel
[root@huangkai data]# yum -y install perl
```
下载 erlang:
```
[root@huangkai data]# wget http://www.erlang.org/download/otp_src_20.2.tar.gz
```
安装erlang:
将 gz文件解压到 `/usr/local/src`，安装在 `/usr/local` 目录下
```
[root@huangkai data]# tar -zxvf otp_src_20.2.tar.gz -C /usr/local/src
[root@huangkai data]# cd /usr/local/src
[root@huangkai src]# ./configure --prefix=/usr/local
[root@huangkai src]# make && make install
```
2、下载 rabbitmq安装包：
```
[root@huangkai data]# https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.3/rabbitmq-server-generic-unix-3.7.3.tar.xz
```

安装 rabbitmq:
下载的是一个rabbitmq-server-generic-unix-3.7.3.tar.xz文件：

.tar.xz的解压方法： 
```
[root@huangkai data]# xz -d rabbitmq-server-generic-unix-3.7.3.tar.xz
``` 
会产生一个mpfr-3.1.2.tar 文件，再执行
``` 
[root@huangkai data]# tar -xvf rabbitmq-server-generic-unix-3.7.3.tar -C /usr/local
```

启动web管理控制台：
``` 
[root@huangkai sbin]#  ./rabbitmq-plugins enable rabbitmq_management
```

添加用户 :
rabbitmq默认的用户为 guest,密码也为guest,只能在本地使用 `localhost`访问，添加一个可以远程访问的使用如下：
``` 
[root@huangkai sbin]# ./rabbitmqctl add_user admin admin
```

设置用户tag :
``` 
[root@huangkai sbin]# ./rabbitmqctl set_user_tags admin administrator
```

重启服务 ：
```
[root@huangkai sbin]# ./rabbitmq-service stop
[root@huangkai sbin]# ./rabbitmq-server start

  ##  ##
  ##  ##      RabbitMQ 3.7.3. Copyright (C) 2007-2018 Pivotal Software, Inc.
  ##########  Licensed under the MPL.  See http://www.rabbitmq.com/
  ######  ##
  ##########  Logs: /usr/local/rabbitmq_server-3.7.3/var/log/rabbitmq/rabbit@huangkai.log
                    /usr/local/rabbitmq_server-3.7.3/var/log/rabbitmq/rabbit@huangkai_upgrade.log

              Starting broker...
 completed with 3 plugins.
```

web访问：http://192.168.1.90:15672
用户名：admin
密码:admin