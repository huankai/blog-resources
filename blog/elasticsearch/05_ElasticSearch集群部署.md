---
title: ElasticSearch 集群部署
date: {{ date }}
author: huangkai
tags:
    - ElasticSearch
---

# 一、部署环境 #

ip: 192.168.1.50 192.168.1.51 192.168.1.90



## 192.168.1.50 ##

 docker-compose.yml 内容：

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
     - "/data/docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml"
     - "/data/docker/elasticsearch/data:/usr/share/elasticsearch/data"
     - "/data/docker/elasticsearch/plugins/ik:/usr/share/elasticsearch/plugins/ik"
    restart: "always"
 ```

 配置文件内容：
 ```
[huangkai@huangkai50 docker]$ cat elasticsearch/config/elasticsearch.yml
#集群名称，三个节点名称要一样 
cluster.name: "docker-cluster"
network.host: 0
# 当前机器公网ip
network.publish_host: 192.168.1.50
# 当前节点名称
node.name: node-1
# minimum_master_nodes need to be explicitly set when bound on a public IP
# # set to 1 to allow single node clusters
# # Details: https://github.com/elastic/elasticsearch/pull/17288
#集群节点信息
discovery.zen.ping.unicast.hosts: ["192.168.1.50","192.168.1.51","192.168.1.90"]
discovery.zen.minimum_master_nodes: 1
[huangkai@huangkai50 docker]$ 
 ```


 ## 192.168.1.51 ##

 docker-compose.yml 内容：

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
     - "/data/docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml"
     - "/data/docker/elasticsearch/data:/usr/share/elasticsearch/data"
     - "/data/docker/elasticsearch/plugins/ik:/usr/share/elasticsearch/plugins/ik"
    restart: "always"
 ```

 配置文件内容：
 ```
[huangkai@huangkai50 docker]$ cat elasticsearch/config/elasticsearch.yml
#集群名称，三个节点名称要一样 
cluster.name: "docker-cluster"
network.host: 0
# 当前机器公网ip
network.publish_host: 192.168.1.51
# 当前节点名称
node.name: node-2
# minimum_master_nodes need to be explicitly set when bound on a public IP
# # set to 1 to allow single node clusters
# # Details: https://github.com/elastic/elasticsearch/pull/17288
#集群节点信息
discovery.zen.ping.unicast.hosts: ["192.168.1.50","192.168.1.51","192.168.1.90"]
discovery.zen.minimum_master_nodes: 1
[huangkai@huangkai50 docker]$ 
 ```




## 192.168.1.90 ##

 docker-compose.yml 内容：

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
     - "/data/docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml"
     - "/data/docker/elasticsearch/data:/usr/share/elasticsearch/data"
     - "/data/docker/elasticsearch/plugins/ik:/usr/share/elasticsearch/plugins/ik"
    restart: "always"
 ```

 配置文件内容：
 ```
[huangkai@huangkai50 docker]$ cat elasticsearch/config/elasticsearch.yml
#集群名称，三个节点名称要一样 
cluster.name: "docker-cluster"
network.host: 0
# 当前机器公网ip
network.publish_host: 192.168.1.90
# 当前节点名称
node.name: node-3
# minimum_master_nodes need to be explicitly set when bound on a public IP
# # set to 1 to allow single node clusters
# # Details: https://github.com/elastic/elasticsearch/pull/17288
#集群节点信息
discovery.zen.ping.unicast.hosts: ["192.168.1.50","192.168.1.51","192.168.1.90"]
discovery.zen.minimum_master_nodes: 1
[huangkai@huangkai50 docker]$ 
 ```

 启动三个节点：
 使用浏览器查看集群信息：
 http://192.168.1.50:9200/_cat/nodes?pretty
 http://192.168.1.51:9200/_cat/nodes?pretty
 http://192.168.1.90:9200/_cat/nodes?pretty

 当任意节点不可用时，elasticsearch 会自动更新节点信息。




