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
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/CAS/01.png)

我这里使用Maven的，下载地址：[https://github.com/apereo/cas-overlay-template](https://github.com/apereo/cas-overlay-template)

# 2、构筑Overlay #
下载下来的Overlay默认配置就可以直接构筑能用的war包，直接使用它下边的build脚本执行 
```
build package
```
第一次构筑比较慢，可以在pom的repositories里加一个ali源，构筑会快一些。
构筑完后在target下找到一个war包，放到你的tomcat8.5（官方建议8.0以上版本，我建议使用8.5，如果你用8.0跑不起来，记得换成8.5以上版本）下跑起来试试吧：

http://localhost:8080/cas/login  默认账号：casuser  默认密码：Mellon  目前的配置仅有这一个用户。

第一次会有两个红色警告，一个就是说你没用HTTPS登录，另一个就是你现在只有一个写死的用户，目前这个服务端只能看看，没什么实际用途。

别急，我们下边开始解决这两个问题。

这里先不要急着删掉你Tomcat下war包刚刚解压出来的内容