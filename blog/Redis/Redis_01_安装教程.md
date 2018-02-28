---
title: Redis安装
date: {{ date }}
author: huangkai
tags:
    - Redis
---

## 安装环境 ##
- 系统版本：CentOS Linux release 7.3.1611 (Core)
- Redis版本：4.0.1

## 下载 ##
下载地址： 请点击 [官网](http://download.redis.io/releases/redis-4.0.1.tar.gz) 下载 ，下载完成后，上传到 Linux某个目录中。

也可以直接使用 wget 命令下载：

```
[huangkai@sjq01 soft]$ wget http://download.redis.io/releases/redis-4.0.1.tar.gz
--2017-10-20 14:10:53--  http://download.redis.io/releases/redis-4.0.1.tar.gz
Resolving download.redis.io (download.redis.io)... 109.74.203.151
Connecting to download.redis.io (download.redis.io)|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1711660 (1.6M) [application/x-gzip]
Saving to: ‘redis-4.0.1.tar.gz’

100%[===============================================================================================================================================================================================>] 1,711,660    585KB/s   in 2.9s   
2017-10-20 14:10:57 (585 KB/s) - ‘redis-4.0.1.tar.gz’ saved [1711660/1711660]
[huangkai@sjq01 soft]$ ll
total 1672
-rw-rw-r--. 1 huangkai huangkai 1711660 Jul 24 21:59 redis-4.0.1.tar.gz
```
```
[huangkai@sjq01 soft]$ wget http://download.redis.io/releases/redis-4.0.1.tar.gz
--2017-10-20 14:10:53--  http://download.redis.io/releases/redis-4.0.1.tar.gz
Resolving download.redis.io (download.redis.io)... 109.74.203.151
Connecting to download.redis.io (download.redis.io)|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1711660 (1.6M) [application/x-gzip]
Saving to: ‘redis-4.0.1.tar.gz’

100%[===============================================================================================================================================================================================>] 1,711,660    585KB/s   in 2.9s   
2017-10-20 14:10:57 (585 KB/s) - ‘redis-4.0.1.tar.gz’ saved [1711660/1711660]
[huangkai@sjq01 soft]$ ll
total 1672
-rw-rw-r--. 1 huangkai huangkai 1711660 Jul 24 21:59 redis-4.0.1.tar.gz
```

## 安装 ##

**1、解压文件**
解压文件到指定目录 (``/usr/local/``)

```
[huangkai@sjq01 soft]$ sudo tar xzf redis-4.0.1.tar.gz -C /usr/local/
[sudo] password for huangkai: 
[huangkai@sjq01 soft]$ cd /usr/local/redis-4.0.1
[huangkai@sjq01 redis-4.0.1]$ ll
total 268
-rw-rw-r--.  1 huangkai huangkai 127778 Jul 24 21:58 00-RELEASENOTES
-rw-rw-r--.  1 huangkai huangkai     53 Jul 24 21:58 BUGS
-rw-rw-r--.  1 huangkai huangkai   1815 Jul 24 21:58 CONTRIBUTING
-rw-rw-r--.  1 huangkai huangkai   1487 Jul 24 21:58 COPYING
drwxrwxr-x.  6 huangkai huangkai    124 Jul 24 21:58 deps
-rw-rw-r--.  1 huangkai huangkai     11 Jul 24 21:58 INSTALL
-rw-rw-r--.  1 huangkai huangkai    151 Jul 24 21:58 Makefile
-rw-rw-r--.  1 huangkai huangkai   4223 Jul 24 21:58 MANIFESTO
-rw-rw-r--.  1 huangkai huangkai  20530 Jul 24 21:58 README.md
-rw-rw-r--.  1 huangkai huangkai  57764 Jul 24 21:58 redis.conf
-rwxrwxr-x.  1 huangkai huangkai    271 Jul 24 21:58 runtest
-rwxrwxr-x.  1 huangkai huangkai    280 Jul 24 21:58 runtest-cluster
-rwxrwxr-x.  1 huangkai huangkai    281 Jul 24 21:58 runtest-sentinel
-rw-rw-r--.  1 huangkai huangkai   7606 Jul 24 21:58 sentinel.conf
drwxrwxr-x.  3 huangkai huangkai   4096 Jul 24 21:58 src
drwxrwxr-x. 10 huangkai huangkai    167 Jul 24 21:58 tests
drwxrwxr-x.  8 huangkai huangkai   4096 Jul 24 21:58 utils
[huangkai@sjq01 redis-4.0.1]$ 

```
如上所示， tar 命令 ``-C`` 参数，表示把 tar.gz包解压到 ``/usr/local``目录中，上面需要输入密码，是因为``/usr/local``目录需要root用户才能访问，此时并不是root用户登陆。
**2、赋予权限(可选操作)**

给当前用户redis操作的权限(可选，如果你安装的目录普通用户有权限访问，或者你使用的是root用户，此操作不需要，如果没有权限，后面的操作可能都会报没有权限的错误)
	将redis-4.0.1目录及其子目录的所有者给 huangkai 

```
[root@sjq01 local]# chown -R huangkai:huangkai redis-4.0.1
[root@sjq01 local]# ll
total 4
drwxr-xr-x. 2 root     root        6 Nov  5  2016 bin
drwxr-xr-x. 2 root     root        6 Nov  5  2016 etc
drwxr-xr-x. 2 root     root        6 Nov  5  2016 games
drwxr-xr-x. 2 root     root        6 Nov  5  2016 include
drwxr-xr-x. 2 root     root        6 Nov  5  2016 lib
drwxr-xr-x. 2 root     root        6 Nov  5  2016 lib64
drwxr-xr-x. 2 root     root        6 Nov  5  2016 libexec
drwxrwxr-x. 6 huangkai huangkai 4096 Jul 24 21:58 redis-4.0.1
drwxr-xr-x. 2 root     root        6 Nov  5  2016 sbin
drwxr-xr-x. 5 root     root       49 Aug 16 09:08 share
drwxr-xr-x. 2 root     root        6 Nov  5  2016 src
```
**3、创建安装目录 **
(bin 和 conf)：
```
[huangkai@sjq01 redis-4.0.1]$ mkdir bin conf
[huangkai@sjq01 redis-4.0.1]$ ll
total 268
-rw-rw-r--.  1 huangkai huangkai 127778 Jul 24 21:58 00-RELEASENOTES
drwxrwxr-x.  2 huangkai huangkai      6 Oct 20 15:36 bin
-rw-rw-r--.  1 huangkai huangkai     53 Jul 24 21:58 BUGS
-rw-rw-r--.  1 huangkai huangkai   1815 Jul 24 21:58 CONTRIBUTING
-rw-rw-r--.  1 huangkai huangkai   1487 Jul 24 21:58 COPYING
drwxrwxr-x.  6 huangkai huangkai    124 Jul 24 21:58 deps
drwxrwxr-x.  2 huangkai huangkai      6 Oct 20 15:36 conf
-rw-rw-r--.  1 huangkai huangkai     11 Jul 24 21:58 INSTALL
-rw-rw-r--.  1 huangkai huangkai    151 Jul 24 21:58 Makefile
-rw-rw-r--.  1 huangkai huangkai   4223 Jul 24 21:58 MANIFESTO
-rw-rw-r--.  1 huangkai huangkai  20530 Jul 24 21:58 README.md
-rw-rw-r--.  1 huangkai huangkai  57764 Jul 24 21:58 redis.conf
-rwxrwxr-x.  1 huangkai huangkai    271 Jul 24 21:58 runtest
-rwxrwxr-x.  1 huangkai huangkai    280 Jul 24 21:58 runtest-cluster
-rwxrwxr-x.  1 huangkai huangkai    281 Jul 24 21:58 runtest-sentinel
-rw-rw-r--.  1 huangkai huangkai   7606 Jul 24 21:58 sentinel.conf
drwxrwxr-x.  3 huangkai huangkai   4096 Jul 24 21:58 src
drwxrwxr-x. 10 huangkai huangkai    167 Jul 24 21:58 tests
drwxrwxr-x.  8 huangkai huangkai   4096 Jul 24 21:58 utils
[huangkai@sjq01 redis-4.0.1]$ 
```
**3、编译安装**

编译:	<font color="red">** make** </font>
编译安装:<font color="red">**sudo make install PREFIX=/usr/local/redis-4.0.1/ **</font> ，会在 ``/usr/local/redis-4.0.1/bin``目录中创建Redis脚本文件
```
[huangkai@sjq01 redis-4.0.1]$ make
----------
一系列编译与安装信息
----------
[huangkai@sjq01 redis-4.0.1]$ sudo make install PREFIX=/usr/local/redis-4.0.1/
----日 志 信 息 开 始------
cd src && make install
make[1]: Entering directory `/usr/local/redis-4.0.1/src'
    CC Makefile.dep
make[1]: Leaving directory `/usr/local/redis-4.0.1/src'
make[1]: Entering directory `/usr/local/redis-4.0.1/src'
Hint: It's a good idea to run 'make test' ;)
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/usr/local/redis-4.0.1/src'
----日 志 信 息 结 束------
[huangkai@sjq01 redis-4.0.1]$ 
[huangkai@sjq01 bin]$ pwd
/usr/local/redis-4.0.1/bin
[huangkai@sjq01 bin]$ ll
total 21768
-rwxr-xr-x. 1 root root 2450880 Oct 20 15:55 redis-benchmark
-rwxr-xr-x. 1 root root 5740808 Oct 20 15:55 redis-check-aof
-rwxr-xr-x. 1 root root 5740808 Oct 20 15:55 redis-check-rdb
-rwxr-xr-x. 1 root root 2605168 Oct 20 15:55 redis-cli
lrwxrwxrwx. 1 root root      12 Oct 20 15:55 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 5740808 Oct 20 15:55 redis-server
[huangkai@sjq01 bin]$
```

**4、复制配置文件**
```
[huangkai@sjq01 redis-4.0.1]$ cp redis.conf ./conf/
[huangkai@sjq01 redis-4.0.1]$ cd conf/
[huangkai@sjq01 conf]$ ll
total 60
-rw-rw-r--. 1 huangkai huangkai 57764 Oct 20 16:02 redis.conf
```

**5、启动服务**

编译完成后二进制文件是在src目录下，通过下面的命令启动Redis服务：
```
[huangkai@sjq01 redis-4.0.1]$ ./bin/redis-server ./conf/redis.conf
5290:C 20 Oct 14:26:05.100 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
5290:C 20 Oct 14:26:05.100 # Redis version=4.0.1, bits=64, commit=00000000, modified=0, pid=5290, just started
5290:C 20 Oct 14:26:05.100 # Warning: no config file specified, using the default config. In order to specify a config file use ./redis-server /path/to/redis.conf
5290:M 20 Oct 14:26:05.101 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
5290:M 20 Oct 14:26:05.101 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
5290:M 20 Oct 14:26:05.101 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.1 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 5290
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

5290:M 20 Oct 14:26:05.103 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
5290:M 20 Oct 14:26:05.103 # Server initialized
5290:M 20 Oct 14:26:05.103 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
5290:M 20 Oct 14:26:05.103 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
5290:M 20 Oct 14:26:05.103 * Ready to accept connections
```

如果控制台出现如上信息，表示Redis已启动成功。
## 连接测试 ##
你可以使用内置的客户端命令redis-cli进行使用，新开一个连接窗口
```
[root@sjq01 ~]# cd /usr/local/redis-4.0.1/bin/
[root@sjq01 bin]# ./redis-cli 
127.0.0.1:6379> SET name huangkai
OK
127.0.0.1:6379> GET name
"huangkai"
127.0.0.1:6379> 
```
## 关闭服务 ##
方式1、使用 <font color="red">**kill**</font> 命令关闭进程(一般不建议此方式)

```
[root@sjq01 bin]# ps aux|grep redis
huangkai   5290  0.3  0.7 145248  7968 pts/0    Sl+  14:26   0:04 ./redis-server *:6379
root       5370  0.0  0.0 112648   964 pts/1    R+   14:50   0:00 grep --color=auto redis
[root@sjq01 src]# kill -9 5290
[root@sjq01 src]# 

```
&nbsp;&nbsp;方式2、优雅关闭Redis
&nbsp;&nbsp;&nbsp;&nbsp;使用 <font color="red">**SHUTDOWN**</font> 命令
```
[root@sjq01 bin]# ./redis-cli 
127.0.0.1:6379> SHUTDOWN
not connected> quit
[root@sjq01 src]# 
```
## WARNING FIX ##
从上面redis启动之后的控制台打出的信息来看，有3条WARNING信息。他们分别如下：

**1、WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.**

解决方式：
```
第一种：无需重启系统即可生效；但重启以后信息丢失
[huangkai@sjq01 redis-4.0.1]$ su - root	#切换到root用户
Password: 
Last login: Fri Oct 20 14:26:20 CST 2017 from 192.168.184.1 on pts/1
[root@sjq01 ~]# echo 511 >/proc/sys/net/core/somaxconn  #修改配置
----------
第二种：即可生效，重启不会丢失
[root@sjq01 ~]# vim /etc/sysctl.conf #打开配置文件，在其后追加
net.core.somaxconn = 511
[root@sjq01 ~]# sysctl -p	#使配置文件sysctl.conf生效
```
原理：
对于一个TCP连接，Server与Client需要通过三次握手来建立网络连接.当三次握手成功后,我们可以看到端口的状态由LISTEN转变为ESTABLISHED,接着这条链路上就可以开始传送数据了.每一个处于监听(Listen)状态的端口,都有自己的监听队列.监听队列的长度,与如下两方面有关：一个是 somaxconn参数；另一个是使用该端口的程序中listen()函数.故而somaxconn会限制了接收新 TCP 连接侦听队列的大小。对于一个经常处理新连接的高负载 web服务环境来说，默认的 128 太小了。大多数环境这个值建议增加到 1024 或者更多。 服务进程会自己限制侦听队列的大小，常常在它们的配置文件中有设置队列大小的选项。大的侦听队列对防止拒绝服务 DoS 攻击也会有所帮助。

redis配置文件中有个参数，tcp-backlog默认值是511，而系统默认的somaxconn是128，所以redis启动以后报出这个warnning

**2、WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.**

解决办法：
```
第一种：无需重启系统即可生效；但重启以后信息丢失
[huangkai@sjq01 redis-4.0.1]$ su - root	#切换到root用户
Password: 
Last login: Fri Oct 20 14:26:20 CST 2017 from 192.168.184.1 on pts/1
[root@sjq01 ~]# echo 1 > /proc/sys/vm/overcommit_memory  #修改配置
----------
第二种：即可生效，重启不会丢失
[root@sjq01 ~]# vim /etc/sysctl.conf #打开配置文件，在其后追加
vm.overcommit_memory=1
[root@sjq01 ~]# sysctl -p	#使配置文件sysctl.conf生效
```
原理：

内核参数overcommit_memory 确定了内存分配策略，可选值为[0、1、2]
0： 表示内核将检查是否有足够的可用内存供应用进程使用；如果有足够的可用内存，内存申请允许；否则，内存申请失败，并把错误返回给应用进程。
1： 表示内核允许分配所有的物理内存，而不管当前的内存状态如何。
2： 表示内核允许分配超过所有物理内存和交换空间总和的内存

 进程通常调用malloc()函数来请求分配内存，Linux支持超量分配内存，以允许分配比可用RAM加上交换内存的请求。Linux对大部分申请内存的请求都回复"yes"，以便能跑更多更大的程序。因为申请内存后，并不会马上使用内存。这种技术叫做Overcommit。当linux发现内存不足时，会发生OOM killer(OOM=out-of-memory)。它会选择杀死一些进程(用户态进程，不是内核线程)，以便释放内存。当oom-killer发生时，linux会选择杀死哪些进程？选择进程的函数是oom_badness函数(在mm/oom_kill.c中)，该函数会计算每个进程的点数(0~1000)。点数越高，这个进程越有可能被杀死。每个进程的点数跟oom_score_adj有关，而且oom_score_adj可以被设置(-1000最低，1000最高)。

**3、WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.**

意思是 你使用的是透明大页，可能导致redis延迟和内存使用问题
解决办法：
```
第一种方法：重启生效，不会丢失
[huangkai@sjq01 redis-4.0.1]$ su - root	#切换到root用户
Password: 
Last login: Fri Oct 20 14:26:20 CST 2017 from 192.168.184.1 on pts/1
[root@sjq01 ~]# vim /etc/rc.d/rc.local #打开配置文件，在其后追加
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi
[root@sjq01 ~]# cat /etc/rc.d/rc.local  #查看修改后的内容 
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.
touch /var/lock/subsys/local
if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi


[root@sjq01 ~]# 
[root@sjq01 ~]# reboot	#重启系统
----------
第二种方法：及时生效，重启丢失
[root@sjq01 ~]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

原理：

正常来说，有两种方式来增加内存，可以管理的内存大小：
1.增大硬件内存管理单元的大小。
2.增大page的大小。
第一个方法不是很现实，现代的硬件内存管理单元最多只支持数百到上千的page表记录，并且，对于数百万page表记录的维护算法必将与目前的数百条记录的维护算法大不相同才能保证性能，目前的解决办法是，如果一个程序所需内存page数量超过了内存管理单元的处理大小，操作系统会采用软件管理的内存管理单元，但这会使程序运行的速度变慢。

从redhat 6（centos，sl，ol）开始，操作系统开始支持 Huge Pages，也就是大页。简单来说， Huge Pages就是大小为2M到1GB的内存page，主要用于管理数千兆的内存，比如1GB的page对于1TB的内存来说是相对比较合适的。

THP（Transparent Huge Pages）是一个使管理Huge Pages自动化的抽象层。目前需要注意的是，由于实现方式问题，THP会造成内存锁影响性能，尤其是在程序不是专门为大内内存页开发的时候，简单介绍如下：操作系统后台有一个叫做khugepaged的进程，它会一直扫描所有进程占用的内存，在可能的情况下会把4kpage交换为Huge Pages，在这个过程中，对于操作的内存的各种分配活动都需要各种内存锁，直接影响程序的内存访问性能，并且，这个过程对于应用是透明的，在应用层面不可控制,对于专门为4k page优化的程序来说，可能会造成随机的性能下降现象。


## 将Redis命令加入环境变量 ##

```
[rootoot@sjq01 bin]# vim + /etc/profile  #打开配置文件，添加如下内容到最后
export PATH=$PATH:/usr/local/redis-4.0.1/bin
[root@sjq01 bin]# source /etc/profile		#使配置文件生效
```
