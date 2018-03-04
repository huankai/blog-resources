---
title: RabbitMQ 安装
date: {{ date }}
author: huangkai
tags:
    - RabbitMQ
---
# 一、简介 #

# 二、安装 #
rabbitMQ需要java环境，这里不再讲述JDK的安装，百度一大把。
## 2.1 、安装erlang : ##
安装erlang需要依赖的库：
```
[root@huangkai data]# yum -y install gcc gcc-c++  m4 ncurses-devel openssl-devel
[root@huangkai data]# yum -y install perl
```
下载 erlang:
```
[root@huangkai data]# wget http://www.erlang.org/download/otp_src_20.2.tar.gz
```
安装erlang，将 gz文件解压到 `/usr/local/src`，安装在 `/usr/local` 目录下
```
[root@huangkai data]# tar -zxvf otp_src_20.2.tar.gz -C /usr/local/src
[root@huangkai data]# cd /usr/local/src
[root@huangkai src]# ./configure --prefix=/usr/local
[root@huangkai src]# make && make install
```
安装过程可能需要几分钟，安装完成后，使用erl 命令查看是否安装成功。
```
[root@huangkai otp_src_20.2]# erl
Erlang/OTP 20 [erts-9.2] [source] [64-bit] [smp:1:1] [ds:1:1:10] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V9.2  (abort with ^G)
1> 
```
如上出现erl环境，表示安装成功，退出erl环境,可输入 `halt() .` 注意，后面有个小数点不能丢掉，小数点与 `hart()`中间有个空格。 
## 2.2、安装 rabbitmq： ##
下载 rabbitmq安装包：
```
[root@huangkai data]# wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.3/rabbitmq-server-generic-unix-3.7.3.tar.xz
```

安装 rabbitmq，下载的是一个rabbitmq-server-generic-unix-3.7.3.tar.xz文件：
.tar.xz的解压方法： 
```
[root@huangkai data]# xz -d rabbitmq-server-generic-unix-3.7.3.tar.xz
``` 
会产生一个rabbitmq-server-generic-unix-3.7.3.tar 文件，再执行
``` 
[root@huangkai data]# tar -xvf rabbitmq-server-generic-unix-3.7.3.tar -C /usr/local
```

开启web管理控制台：
``` 
[root@huangkai sbin]#  ./rabbitmq-plugins enable rabbitmq_management
```
添加用户：
添加用户时，一定要使用启动rabbitmq服务的系统用户添加，否则可能会添加不成功。
rabbitmq默认的用户为 guest,密码也为guest,只能在本地使用 `localhost`访问，添加一个可以远程访问的使用如下：
语法为： `rabbitmqctl add_user 用户名 密码`
``` 
[root@huangkai sbin]# ./rabbitmqctl add_user admin admin
Adding user "admin" ...
```
设置用户 admin 的tag 为 administrator:
语法为：`rabbitmqctl set_user_tags 用户名 [tag...]`
``` 
[root@huangkai sbin]# ./rabbitmqctl set_user_tags admin administrator
Setting tags for user "admin" to [administrator] ...
```
重启服务 ：
```
[root@huangkai sbin]# ./rabbitmq-server stop
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
如上，出现以上信息，表示rabbitmq启动成功，可以使用浏览器访问：``http://ip:15672``
登陆用户名：admin
登陆密码:admin

后台进程启动：
```
[root@huangkai sbin]# ./rabbitmq-server -detached
```