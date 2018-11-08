---
title: Solr简介与安装
date: {{ date }}
author: huangkai
tags:
    - Solr
---


# 一、solr 安装 #

下载：
```
[huangkai@sjq-01 ~]$ wget http://archive.apache.org/dist/lucene/solr/7.3.1/solr-7.3.1.tgz
```

解压：

```
[huangkai@sjq-01 solr]$ tar -xvf solr-7.3.1.tgz -C /usr/local
```

添加环境变量:

```
[huangkai@sjq-01 solr]$ vim /etc/profile

# 添加如下内容
#solr env
SOLR_HOME=/usr/local/solr-7.3.1
export PATH=$SOLR_HOME/bin:$PATH

huangkai@sjq-01 solr]$ source /etc/profile
```
启动 solr:

```
[huangkai@sjq-01 solr]$ solr start
```

重启 solr:

```
[huangkai@sjq-01 solr]$ solr restart
```

停止 solr:

```
[huangkai@sjq-01 solr]$ solr stop
```

# 二、solr创建solrCore #

## 2.1、使用 solr命令安装： ##

```
[huangkai@sjq-01 solr]$ solr create -c solrCoreName
```
执行如上命令，会在${solr_home}/server/solr/ 目录下创建 solrCoreName的目录，这就是一个solrCore.


## 2.2、使用solr管理界面安装 ##
如在 /data/solr 目录下创建 test3 solrCore

创建目录 ：
```
[huangkai@sjq-01 solr]$ mkdir -p /data/solr/test3
```
将 ${solr_home}/server/configsets/_default/conf 目录复制到 /data/solr/test3 目录下

```
[huangkai@sjq-01 solr]$ cp -R ${solr_home}/server/configsets/_default/conf /data/solr/test3
```

使用管理台创建solrCore
http://${ip}:8983 访问solr web管理台，在 Core Admin 点击 Add Core

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/solr/solr_01.png)

参数说明：
name: solrCore 的名称，在这里就是test3
instanceDir: solrCore目录
dataDir: solrCore 数据目录
config: solr config 文件，在 ${solrCore}/conf目录下
schema: solr schema.xml 文件，在 ${solrCore}/conf目录下

# 二、删除solrCore #

```
[huangkai@sjq-01 solr]$ solr delete -c solrCoreName
```

参数说明：
-c 参数指定 solrCore的名称