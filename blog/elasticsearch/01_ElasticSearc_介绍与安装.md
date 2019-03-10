---
title: ElasticSearch 简介与安装
date: {{ date }}
author: huangkai
tags:
    - ElasticSearch
---

# 一、 介绍 #


# 二、 安装 #

## 2.1、使用 docker ##

ip:  192.168.1.50

docker-compose.yml 内容如下：

```
elasticsearch:
    image: 192.168.1.50:8870/elasticsearch:6.4.3
    container_name: elasticsearch
    ports:
     - "9200:9200"
     - "9300:9300"
    volumes:
     - "/etc/timezone:/etc/timezone:ro"
     - "/etc/localtime:/etc/localtime:ro"
    restart: "always"
```

使用 root 账号修改 linux 参数，添加如下内容，并执行 `sysctl -p` 使配置生效：
```
[root@huangkai50 ~]# vim /etc/sysctl.conf
vm.max_map_count=655360
[root@huangkai50 ~]# sysctl -p
```

启动服务：

```
[huangkai@huangkai50 docker]$ docker-compose up -d elasticsearch
```

启动后，使用浏览器访问 http://192.168.1.50:9200/ ，如果能正常响应，elasticsearch 启动成功 。

ElasticSearch 详细文档：
https://www.elastic.co
https://es.xiaoleilu.com/010_Intro/10_Installing_ES.html



