---
title: Linux 配置

date: {{ date }}

author: huangkai

tags:
    - Linux
---

# 一、配置网络环境 #

[huangkai@sjq150 hadoop]$ vim /etc/sysconfig/network-scripts/ifcfg-ens33

```
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
UUID="746efeb2-2501-434f-8589-3b3539aaecc2"
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.64.128
PREFIX=24
GATEWAY=192.168.64.2
DNS1=192.168.64.2
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_PRIVACY=no

```

# 二、修改运行级别 #

```
systemctl set-default multi-user.target #设置运行级别为 3，命令行界面
systemctl set-default graphical.target # 设置运行级别为 5 ，图形化界面
```

# 三、关闭 SELINUX #

 SElinux是强制访问控制(MAC)安全系统，是linux历史上最杰出的新安全系统
SELINUX 状态：
- enforcing 强制模式
- permissive 警告模式
- disabled 关闭

```
[huangkai@sjq150 hadoop]$ vim /etc/sysconfig/selinux
SELINUX=disabled
```

# 四、配置 SSH 限制IP 登陆 #

# 五、配置服务器时间与网络时间同步 #

```
[root@localhost ~]# date
Wed Jun 19 10: 59: 26 CST 2019
[root@localhost ~]# yum -y install ntpdate
[root@localhost ~]# ntpdate time.nuri.net
19 Jun 11: 00: 33 ntpdate[32264]: step time server 211.115.194.21 offset 57.200854 sec
[root@localhost ~]# date
Wed Jun 19 11: 00: 42 CST 2019
[root@localhost ~]# 
```

