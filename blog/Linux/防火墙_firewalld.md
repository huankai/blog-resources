---
title: Centos 防火墙(firewalld)
date: {{ date }}
author: huangkai
tags:
    - Linux
---


开机启动防火墙
```
[root@sjq-05 ~]# systemctl enable firewalld
```

查看防火墙状态：
```
[root@sjq-05 ~]# systemctl status firewalld
```

启动防火墙：
```
[root@sjq-05 ~]# systemctl start firewalld
```

关闭防火墙：
```
[root@sjq-05 ~]# systemctl stop firewalld
```
  
开放 80 端口
```
[root@sjq-05 ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
```

重新加载防火墙配置：
```
[root@sjq-05 ~]# firewall-cmd --reload
```

查看开放端口:	
```
[root@sjq-05 ~]# firewall-cmd --zone=public --list-ports
```