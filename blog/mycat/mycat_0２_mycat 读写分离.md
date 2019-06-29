---
title: spring－boot－data－jpa 与 mycat整合
date: {{ date }}
author: huangkai
tags: 
	- mycat
---

## 一、配置主从 ##


|ip|描述|
|::|::|
|192.168.117.100|mysql master|
|192.168.117.101|mysql slave|
|192.168.117.102|mycat|

mysql 主从配置，请查看 mysql 主从配置篇
mycat 安装，请查看 Mycat 安装教程

## 二、配置spring data jpa ##
spring boot 版本：2.1.3.RELEASE
mysql 版本： 8.0.15


spring data jpa 数据库连接配置如下
application.yml:
```
spring:
  jpa:
    show-sql: false
    generate-ddl: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        enable_lazy_load_no_trans: true
  ######################################################### datasource 配置
  datasource:
    name: hikari
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://192.168.117.102:8066/hk_pms?characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
    username: root
    password: 123456
    hikari:
      maximum-pool-size: 50
      max-lifetime: 600000
      minimum-idle: 10
      connection-timeout: 30000
      read-only: false
      connection-test-query: SELECT 1
pagehelper:
  reasonable: false
  support-methods-arguments: true
  params: count=countSql
mybatis:
  mapper-locations:
    - classpath:mybatis/mappers/*.xml
  configuration:
    map-underscore-to-camel-case: true

```

## 二、 spring data jpa 改进##
查看该源码：
https://github.com/huankai/hk-core/blob/master/hk-core-data-jpa/src/main/java/com/hk/core/data/jpa/repository/BaseSimpleJpaRepository.java

## 三、测试 ##


