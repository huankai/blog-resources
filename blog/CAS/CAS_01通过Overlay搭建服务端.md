---
title: CAS 单点登陆(通过Overlay搭建服务端)
date: {{ date }}
author: huangkai
tags:
categories:
    - CAS
---

本系列教程使用的CAS版本为 5.2.0-RC4，官方文档 请点击 [这里](https://apereo.github.io/cas/5.1.x/index.html)
需要下载 [JDK1.8](http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html) 、[Tomcat8.5](https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.20/bin/) 、[Maven3.3](https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/) ，按需要的版本下载即可。

# 1、下载 Overlay #

通过阅读官网文档（[https://apereo.github.io/cas/5.1.x/planning/Getting-Started.html](https://apereo.github.io/cas/5.1.x/planning/Getting-Started.html)）了解到官方建议我们：
```
It is recommended to build and deploy CAS locally using the WAR Overlay method. 
```
通过使用一个名叫Overlay的项目来生成一个可以直接用的war包，来部署服务端，于是我们先下载这个项目，官网给出了两个构筑格式的：

