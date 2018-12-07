---
title: Solr 配置中文分词器
date: {{ date }}
author: huangkai
tags:
    - Solr
---

# 一、配置 smartcn 中文分词器： #

## 1.1、复制中文分词器包 ##

进入docker容器，把smartcn中文分词器架包复制到solr项目的WEB-INF/lib目录下：

```
solr@02f71335fa11:/opt/solr$ cp /opt/solr/contrib/analysis-extras/lucene-libs/lucene-analyzers-smartcn-7.4.0.jar /opt/solr/server/solr-webapp/webapp/WEB-INF/lib/
```

## 1.2、修改配置文件 ##

修改 solrhome/${collection}/conf/managed-schema

```
[huangkai@localhost conf]$ sudo vim /data/docker/solr/server/solr/mycores/mysolr/conf/managed-schema

# 添加如下内容
<fieldType name="text_smartcn" class="solr.TextField" positionIncrementGap="100">
	<analyzer type="index">
  		<tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
	</analyzer>
	<analyzer type="query">
  		<tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
	</analyzer>
</fieldType>
```

## 1.3、重新启动服务： ##

```
[huangkai@localhost docker]$ docker-compose restart solr
```

## 1.4、查看效果 ##

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/solr/solr_03.png)