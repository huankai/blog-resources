---
title: Nginx Linux 安装
date: {{ date }}
author: huangkai
tags:
    - Nginx
    - Linux
---
# 一、什么是 nginx? #
&nbsp;&nbsp;&nbsp;&nbsp;nginx是一种高性能的HTTP和反向代理服务器 ，同时也是一个代理邮件服务器，我们在nginx上可以发布网站，也可以实现负载均衡的功能 ，还可以作为邮件服务器实现收发邮件等功能 。所谓负载均衡是指当同时有多个用户访问服务器的时候，为了减轻服务器压力，我们需要将用户分别引入各服务器，分担服务器的压力。
&nbsp;&nbsp;&nbsp;&nbsp;nginx 可以实现高并发、部署简单、 内存消耗少、成本低等优点。

# 二、nginx 安装 #
- 下载
点击 [此处](http://nginx.org/en/download.html) 下载
下载完后，将软件包上传到linux指定目录（也可以使用 wget下载，如下）：
```
[root@huangkai200 src]# wget http://nginx.org/download/nginx-1.10.3.tar.gz
[root@huangkai200 src]# pwd
/usr/src
[root@huangkai200 src]# ll
total 910812
-rw-r--r--. 1 root root     910812 Dec 24 21:45 nginx-1.10.3.tar.gz
```
- 添加用户
```
[root@huangkai200 src]# useradd nginx -s /sbin/nologin -M	
-s 表示指定用户登陆所使用的 shell，nologin表示此用户不能登陆 
-M 表示不创建用户home目录
```

- 安装

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在安装过程中，可能会遇到一些错误
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nginx 安装需要 openssl、gcc、pcre  的支持
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可先执行如下命令：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">**yum install openssl openssl-devel -y**</font>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">**yum install pcre pcre-devel gcc-c++ -y**</font>

```
[root@huangkai200 src]# tar -zxvf nginx-1.10.3.tar.gz		# 解压文件
[root@huangkai200 src]# ./configure --user=nginx --group=nginx --prefix=/usr/local/soft/nginx --with-http_stub_status_module --with-http_ssl_module
--user 	  指定用户
--group  	指定组
--prefix 	指定安装路径
--with-http_stub_status_model    激活状态信息
--with-http_ssl_module      激活SSL功能   
[root@huangkai200 src]# make
[root@huangkai200 src]# makeinstall
```
- nginx 查看版本

&nbsp;&nbsp;&nbsp;&nbsp;进入nginx根目录 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color="red">${nginx}/sbin/nginx -v</font>

# 三、启动、关闭、重启 nginx #
nginx 默认端口号为 80 ,在 Linux 中，默认 0 —— 1024 号端口需要 root用户才能启动，所以需要切换到 root用户，或者更改nginx 默认端口号。
```
[huangkai@huangkai200 sbin]$ su - root  # 切换到root用户
Password: 
Last login: Sat Mar 18 15:02:28 CST 2017 on pts/0
[root@huangkai200 ~]# cd /usr/local/soft/nginx/
[root@huangkai200 nginx]# ./sbin/nginx -t # 检查nginx 配置
nginx: the configuration file /usr/local/soft/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/soft/nginx/conf/nginx.conf test is successful
```

- 启动nginx
```
[root@huangkai200 nginx]# ./sbin/nginx 
[root@huangkai200 nginx]# ps -ef|grep nginx
gdm       1338  1218  0 13:21 ?        00:00:00 /usr/libexec/ibus-engine-simple
root      6597     1  0 15:20 ?        00:00:00 nginx: master process ./sbin/nginx
nginx     6598  6597  0 15:20 ?        00:00:00 nginx: worker process
root      6610  6474  0 15:20 pts/0    00:00:00 grep --color=auto ngin
```
如上，表示 nginx 启动成功，确保防火墙打开或开放 80 端口的情况下，使用浏览器访问 http://ip地址

- nginx 平滑启动
进入 ${nginx}/sbin目录，执行 <font color="red">**./sbin nginx -s reload**</font> ，平滑启动必须是nginx已运行状态

- nginx 停止
 进入 ${nginx}/sbin目录，执行 <font color="red">**./nginx stop**</font>

