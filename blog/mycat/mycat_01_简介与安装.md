---
title: mycat 简介与安装
date: {{ date }}
author: huangkai
tags: 
	- mycat
---

# 一 、介绍 #

请找度娘 ...

## 二、安装 ##

mycat 在安装前需要配置 jre 环境,请找度娘...
## 1、下载地址 ##

根据自己的要求下载相应的版本: http://dl.mycat.io/

```
[huangkai@localhost soft]$ pwd
/www/soft
[huangkai@localhost soft]$ wget http://dl.mycat.io/1.6.5/Mycat-server-1.6.5-release-20180122220033-linux.tar.gz
```

## 2、解压与配置 ##

```
[huangkai@localhost soft]$ ll
-rw-rw-r--. 1 huangkai huangkai 17480035 Jan 22  2018 Mycat-server-1.6.5-release-20180122220033-linux.tar.gz
[huangkai@localhost soft]$ tar -xvf Mycat-server-1.6.5-release-20180122220033-linux.tar.gz

[huangkai@localhost soft]$ sudo vim + /etc/profile  # 配置环境变量,在最后添加如下两行
export MYCAT_HOME=/www/soft/mycat
export PATH=$MYCAT_HOME/bin:$PATH
[huangkai@localhost soft]$ sudo source /etc/profile
```

## 3、服务命令 ##

**启动:**

如果你使用的是 mysql8 ，请将 ${MYCAT_HOME}/lib/mysql-connector-java-5.1.35.jar 替换为 mysql 8 对应的驱动包。
启动命令如下：
```
[huangkai@localhost soft]$ mycat start
Starting Mycat-server...
```
**重启:**
 
```
[huangkai@localhost soft]$mycat restart
Stopping Mycat-server...
Stopped Mycat-server.
Starting Mycat-server...
```

**关闭:**

```
[huangkai@localhost soft]$ mycat stop
Stopping Mycat-server...
Stopped Mycat-server.
```

## 4、mycat 目录说明 ##

```
[huangkai@localhost mycat]$ ll
total 12
drwxrwxr-x. 2 huangkai huangkai  190 Jun  3 10:14 bin
drwxrwxr-x. 2 huangkai huangkai    6 Mar  1  2016 catlet
drwxrwxr-x. 4 huangkai huangkai 4096 Jun  3 11:03 conf
drwxrwxr-x. 2 huangkai huangkai 4096 Jun  3 10:17 lib
drwxrwxr-x. 3 huangkai huangkai   74 Jun  3 11:03 logs
drwxr-xr-x. 2 huangkai huangkai   25 Jun  3 10:24 tmlogs
-rwxrwxr-x. 1 huangkai huangkai  219 Jan 22  2018 version.txt
```

 - bin  
 	二进制文件，包括启动、重启、停止服务
 - catlet
 	
 - conf
 	配置文件目录，核心配置文件`schema.xml` 、 `server.xml` 、 `rule.xml`
	- schema.xml 配置逻辑库、逻辑表、物理库与物理表的关系，配置分片，读写等
	- server.xml 配置 mycat 信息，如用户名、属性等。
	- rule.xml 配置分片规则
 - lib
 	依赖包目录
 - logs
 	日志目录
 - tmlogs
 	日志目录
 - version.txt
 	版本信息

