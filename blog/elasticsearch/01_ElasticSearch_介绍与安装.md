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
     - "/www/docker/elasticsearch/config/:/usr/share/elasticsearch/config/"
     - "/www/docker/elasticsearch/data:/usr/share/elasticsearch/data"
     - "/www/docker/elasticsearch/plugins/ik:/usr/share/elasticsearch/plugins/ik"
    restart: "always"
```

elasticsearch.yml 配置如下：
```
# 集群名称，所有加入该集群的名称都必须一致
cluster.name: "docker-cluster"

# 绑定 host,这里绑定所有
network.host: 0

network.publish_host: 192.168.1.50

#集群节点名
node.name: node-1

# http 服务端口，默认 9200
http.port: 9200

# minimum_master_nodes need to be explicitly set when bound on a public IP
# # # set to 1 to allow single node clusters
# # # Details: https://github.com/elastic/elasticsearch/pull/17288

# 配置集群主机地址，单节点不需要配置
# discovery.zen.ping.unicast.hosts: ["192.168.1.100","192.168.1.101","192.168.1.102"]

# 
# discovery.zen.minimum_master_nodes: 2
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



