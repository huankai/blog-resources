---
title: ElasticSearch 可视化工具 Kibana 使用
date: {{ date }}
author: huangkai
tags:
    - ElasticSearch
---


# 一、 介绍 #


# 二、 安装 #

## 2.1、使用 docker ##

docker-compose.yml 内容如下：

```
kibana:
    image: 192.168.1.50:8870/kibana:6.4.3
    container_name: kibana
    ports:
     - "5601:5601"
    volumes:
      - "/etc/timezone:/etc/timezone:ro"
      - "/etc/localtime:/etc/localtime:ro"
      - "/data/docker/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml"
    restart: "always"
```
kibana.yml 内容如下: 
```
[huangkai@instance-3knenqmy docker]$ cat kibana/config/kibana.yml 
---
# Default Kibana configuration from kibana-docker.
#
server.name: kibana
server.host: "0"
#配置上下文路径，默认为 /
server.basePath: /kibana
# 配置上下文路径 是否重新，默认为false，当配置了 server.basePath 后，此值要配置为 true,否则会报 404.
server.rewriteBasePath: true
# 配置国际化，支持 en/zh-CN，默认为 en,配置为 zh-CN时，需要下载汉化插件： https://github.com/anbai-inc/Kibana_Hanization
#i18n.locale: "zh-CN"
elasticsearch.url: http://elasticsearch:9200
xpack.monitoring.ui.container.elasticsearch.enabled: true
```

说明：
 kibana 连接 ElasticSearch 的配置文件为docker 容器中的 `/usr/share/kibana/config/kibana.yml` 中 `elasticsearch.url` 属性值，默认值为 `http://elasticsearch:9200`，因为我这里部署的 kibana 与 elasticsearch 在同一台服务器，所以没有将 kibana的配置文件挂在到宿主机中。

 启动成功后，使用浏览器访问  http://192.168.1.50:5601 即可。


# 三、kibana 基本使用 #

在 kibana 的 Dev Tools 菜单 console 中使用如下语法查看：
```
GET _search
{
  "query": {
    "match_all": {}
  }
}

### 根据Id 查询
## 语法：/索引名称/索引类型/要查询的id
GET /test/commodity/dec3dceed7214a9184ba9cd58da5c4c6

### 添加数据
### 语法： /索引名称/索引类型/id
PUT /test/commodity/2323
{
  "name":"小米",
  "price":2399
}

### 查询所有
### 语法：/索引名称/索引类型/要查询的id
GET /test/commodity/_search


### 根据多个id 查询
GET /test/commodity/_mget
{
  "ids":["dec3dceed7214a9184ba9cd58da5c4c6","bf3727cb6822420c82ff371858f05c16"]
}

### 条件查询：查询 price 为 5288
GET /test/commodity/_search?q=price:5288

### 区间查询：查询价格在 5000 ~ 5500
GET /test/commodity/_search?q=price[5000 TO 5500]

### 排序: 价格 desc | asc
GET /test/commodity/_search?sort=price:asc

### 分页查询
GET /test/commodity/_search?from=0&size=2

###-------- DSL 查询 -------
### term 查询：精确匹配
GET /test/commodity/_search
{
  "query":{
    "term": {
      "price": "5888"
    }
  }
}

### like 查询，支持分词查询(分将参数分词),form和 size ###分页面参数
GET /test/commodity/_search
{
  
  "from": 0, 
  "size": 20, 
  "query":{
    "match": {
      "name": "果"
    }
  }
}
```

kibana 文档：
https://www.elastic.co/products/stack/monitoring
