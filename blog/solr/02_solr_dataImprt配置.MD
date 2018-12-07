---
title: Solr dataImport 配置
date: {{ date }}
author: huangkai
tags:
    - Solr
---

**配置 mysolr dataImport:**

1、在宿主机目录修改`/data/docker/solr/server/solr/mycores/mysolr/conf/solrconfig.xml`,在文件最后添加如下内容:

```
 <requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
    <lst name="defaults">
      <str name="config">data-config.xml</str>
    </lst>
</requestHandler>
```

在宿主机目录 `/data/docker/solr/server/solr/mycores/mysolr/conf/` 添加 `data-config.xml` 文件:
```
<?xml version="1.0" encoding="UTF-8" ?>
<dataConfig>
    <dataSource type="JdbcDataSource"
     driver="com.mysql.cj.jdbc.Driver"
     url="jdbc:mysql://172.17.0.1:3306/hk_pms?characterEncoding=utf8&amp;useSSL=false&amp;serverTimezone=Asia/Shanghai&amp;allowPublicKeyRetrieval=true"
     user="pms"
     password="123456"/>
        <document>
            <entity name="app" query="SELECT * FROM sys_app">
                <field column="id" name="id"/>
                <field column="app_code" name="appCode"/>
                <field column="app_name" name="appName"/>
            </entity>
        </document>
</dataConfig>
```

在宿主机修改 `/data/docker/solr/server/solr/mycores/mysolr/conf/managed-schema` 文件，添加如下内容，注意：该文件中默认的主键字段就是 id，所以id 不再需要添加了：

```
<field name="appCode" type="text_general" indexed="true" stored="true"/>
<field name="appName" type="text_general" indexed="true" stored="true"/>
```
在宿主机修改 `/data/docker/solr/server/solr/mycores/mysolr` 目录下创建 `lib`文件夹，并将 Mysql 的驱动包放在此目录中：

```
[huangkai@localhost mysolr]$ sudo mkdir lib
[huangkai@localhost mysolr]$ ll
total 16
drwxr-xr-x 3 8983 8983 4096 Dec  3 11:00 conf
-rw-r--r-- 1 8983 8983  166 Dec  3 09:54 core.properties
drwxr-xr-x 5 8983 8983 4096 Dec  3 09:54 data
drwxr-xr-x 2 root root 4096 Dec  3 10:57 lib
[huangkai@localhost mysolr]$ ll lib/
total 2084
-rw-rw-r-- 1 huangkai huangkai 2132635 Nov  8 14:35 mysql-connector-java-8.0.13.jar
[huangkai@localhost mysolr]$ 
```

进入docker 容器复制文件：


```
solr@ba01bbd16665:/opt/solr$ cp dist/solr-dataimporthandler-* server/solr-webapp/webapp/WEB-INF/lib/
```

重启 solr:

```
[huangkai@localhost docker]$ docker-compose restart solr
```
