---
title: rocketmq_01介绍与安装
date: {{ date }}
author: huangkai
tags:
    - Rocketmq
---

一、介绍
====


参考资料: [https://github.com/apache/rocketmq-spring/blob/master/README_zh_CN.md](https://github.com/apache/rocketmq-spring/blob/master/README_zh_CN.md)

二、安装
====

2.1、 使用 docker 安装:

docker-compose.yml 内容如下:

```
version: '3.7'
services:
  rmqnamesrv:
    image: foxiswho/rocketmq:server-4.5.2
    container_name: rmqnamesrv
    ports:
      - 9876:9876
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/www/docker/rocketmq/namesrv/logs:/home/rocketmq/logs"
      - "/www/docker/rocketmq/namesrv/conf:/home/rocketmq/rocketmq-4.5.2/conf"
    command: "mqnamesrv"
  	restart: "always"
  rmqbroker:
    image: foxiswho/rocketmq:broker-4.5.2
    container_name: rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/www/docker/rocketmq/broker/logs:/home/rocketmq/logs"
      - "/www/docker/rocketmq/broker/store:/home/rocketmq/store"
      - "/www/docker/rocketmq/broker/conf/broker.conf:/etc/rocketmq/broker.conf"
    environment:
      NAMESRV_ADDR: "rmqnamesrv:9876"
      JAVA_OPT_EXT: "-server -Xms128m -Xmx128m -Xmn128m"
    depends_on:
      - rmqnamesrv
    restart: "always" 
  rmqconsole:
    build: ./rocketmq/console
    container_name: rmqconsole
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/www/docker/rocketmq/console/logs:/app/logs"
      - "/www/docker/rocketmq/console/conf/application.properties:/app/application.properties"
      - "/www/docker/rocketmq/console/rmqconsole:/app/rmqconsole"
    ports:
      - 8080:8080
    depends_on:
      - rmqnamesrv
    restart: "always"
```

2.2、目录结构与权限如下:


```
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ pwd
/www/docker
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ ll
total 44
-rw-rw-r--  1 huangkai huangkai 4744 Nov 15 16:54 docker-compose.yml
drwxrwxr-x  5 huangkai huangkai 4096 Nov 14 17:36 rocketmq
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ cd rocketmq/
[huangkai@iZwz93os69edbdeohzoxjeZ rocketmq]$ ll
total 12
drwxrwxr-x 5     3000     3000 4096 Nov 15 16:36 broker
drwxrwxr-x 5 huangkai huangkai 4096 Nov 14 17:51 console
drwxrwxr-x 4     3000     3000 4096 Nov 15 16:35 namesrv
[huangkai@iZwz93os69edbdeohzoxjeZ rocketmq]$ 
```
上面注意点:
 broker 与 namesrv 在 docker 容器中使用的是 rocketmq 用户启动的，该用户的 pid 为 3000,所以在宿主机上，需要创建一个 3000的用户，或者将该目录的所有者直接改为 3000，`chown 3000:3000 ./broker -R` 、 `chown 3000:3000 ./namesrv -R` 否则，会启动不成功