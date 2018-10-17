---
title: Zookeeper_01介绍与安装
date: {{ date }}
author: huangkai
tags:
    - Zookeeper
---

# 一、Zookeeper 简介 #


# 二、Zookeeper 安装 #
集群安装，请找度娘.


# 三、Zookeeper 配置开机启动 #

```
将zookeeper目录所有者给指定用户 huangkai
[huangkai@sjq-01 ~]$ chown huangkai:huangkai /usr/local/zookeeper-3.4.9/ -R
配置zookeeper 目录当前用户可执行权限
[huangkai@sjq-01 ~]$ chmod u+x /usr/local/zookeeper-3.4.9/ -R
```

配置zookeeper日志目录 ：


1、修改 zoo.cfg 配置，添加内容如下：

```
dataLogDir=/usr/local/zookeeper-3.4.9/logs
```
2、创建日志目录 ：

```
[huangkai@sjq-01 zookeeper-3.4.9]$ mkdir logs
```

3、[huangkai@sjq-01 zookeeper-3.4.9]$ vim ./bin/zkServer.sh

```
....
# 大约在 125行左右：添加如下一句话：
ZOO_LOG_DIR="$($GREP "^[[:space:]]*dataLogDir" "$ZOOCFG" | sed -e 's/.*=//')"


if [ ! -w "$ZOO_LOG_DIR" ] ; then
mkdir -p "$ZOO_LOG_DIR"
fi

_ZOO_DAEMON_OUT="$ZOO_LOG_DIR/zookeeper.out"
....

```

编写 zookeeper.service文件如下：

```
[Unit]
Description=zookeeper
After=zookeeper.service

[Service]
Type=forking
User=huangkai
Group=huangkai
PIDFile=/usr/local/zookeeper-3.4.9/data/zookeeper_server.pid
ExecStart=/usr/local/zookeeper-3.4.9/bin/zkServer.sh start
ExecReload=/usr/local/zookeeper-3.4.9/bin/zkServer.sh restart
ExecStop=/usr/local/zookeeper-3.4.9/bin/zkServer.sh stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
# 三、zookeeper-dev-ZooInspector #
