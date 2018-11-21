---
title: Docker_04_安装redis
date: {{ date }}
author: huangkai
tags:
    - Docker
---

# 一、下载 #

```
[huangkai@sjq-20 ~]$ docker pull redis:5.0
```
# 二、安装 #

## 2.1、创建配置文件： ##


```
[huangkai@sjq-20 ~] mkdir /data/docker/redis5.0

[huangkai@sjq-20 ~] cd /data/docker/redis5.0

[huangkai@sjq-20 redis5.0] mkdir conf data

# 复制 redis.conf 到conf 目录
[huangkai@sjq-20 redis5.0] vim conf/redis.conf

# 后面的 172.17.0.4 是docker 中的redis容器的 ip，可以使用 【docker inspect 容器名】 查看 IPAddrss 参数值
bind 127.0.0.1  172.17.0.4

# 开启保护模式
protected-mode yes

daemonize no

# 其它参数按需要配置
....
```

## 2.2、启动docker : ##

```
[huangkai@sjq-20 ~] docker run -d -p 6379:6379 --restart=always -v /data/docker/redis5.0/conf/redis.conf:/usr/local/etc/redis/redis.conf -v /data/docker/redis5.0/data:/data --name redis5.0 docker.io/redis:5.0 redis-server /usr/local/etc/redis/redis.conf
```
参数说明 
-d: 守护启动
-p: 宿主机与docker 容器端口映射
-v: 挂载目录
--name: docker 容器名称



## 2.3、查看docker容器日志： ##

语法：docker logs 容器名称
```
[huangkai@sjq-20 ~] docker logs redis.50
```


## 2.4、查看docker容器信息： ##

语法：docker inspect 容器名称
```
[huangkai@sjq-20 redis5.0]$ docker inspect redis5.0
[
    {
        "Id": "0a1b92c338dcee20b881cdd23e94bc71780d09ee36dcbe735350edd561783e2a",
        "Created": "2018-11-16T06:24:58.046343465Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "redis-server",
            "/usr/local/etc/redis/redis.conf"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 49541,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2018-11-16T06:24:58.320257898Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:415381a6cb813ef0972eff8edac32069637b4546349d9ffdb8e4f641f55edcdd",
        "ResolvConfPath": "/var/lib/docker/containers/0a1b92c338dcee20b881cdd23e94bc71780d09ee36dcbe735350edd561783e2a/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/0a1b92c338dcee20b881cdd23e94bc71780d09ee36dcbe735350edd561783e2a/hostname",
        "HostsPath": "/var/lib/docker/containers/0a1b92c338dcee20b881cdd23e94bc71780d09ee36dcbe735350edd561783e2a/hosts",
        "LogPath": "/var/lib/docker/containers/0a1b92c338dcee20b881cdd23e94bc71780d09ee36dcbe735350edd561783e2a/0a1b92c338dcee20b881cdd23e94bc71780d09ee36dcbe735350edd561783e2a-json.log",
        "Name": "/redis5.0",
        "RestartCount": 0,
        "Driver": "overlay2",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": [
                "/data/docker/redis5.0/conf/redis.conf:/usr/local/etc/redis/redis.conf",
                "/data/docker/redis5.0/data:/data"
            ],
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "6379/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "6379"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "docker-runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Name": "overlay2",
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/2b13adde61038fc6d23fd1b3e22173b2fdea3db30e6feccfe25ca6fc7bbc14ee-init/diff:/var/lib/docker/overlay2/9778d973fc1298357fd7fd9db3c1171edab74f403dedada41ad82ad1e0ae37c5/diff:/var/lib/docker/overlay2/3e95599c0bf8570df0a44e532be77768563481a8c377e6689e410fe995a1cf3a/diff:/var/lib/docker/overlay2/a64b7cde6f7280ac056a26f1524d270b252058b16024a4e2c663eb000dbbb8df/diff:/var/lib/docker/overlay2/ac877e9edd3b1fd602a5d41037e3a6990ba8d8cf90fbf239e6cad4a2aad99d49/diff:/var/lib/docker/overlay2/f97524bea89ee8b83df4b951052c30c2212d22632e939d244b67d24b34fb57af/diff:/var/lib/docker/overlay2/50b082840fde5861f1c612bc5376dd270e1677ebcd2e81ff9d19541b651eb1b5/diff",
                "MergedDir": "/var/lib/docker/overlay2/2b13adde61038fc6d23fd1b3e22173b2fdea3db30e6feccfe25ca6fc7bbc14ee/merged",
                "UpperDir": "/var/lib/docker/overlay2/2b13adde61038fc6d23fd1b3e22173b2fdea3db30e6feccfe25ca6fc7bbc14ee/diff",
                "WorkDir": "/var/lib/docker/overlay2/2b13adde61038fc6d23fd1b3e22173b2fdea3db30e6feccfe25ca6fc7bbc14ee/work"
            }
        },
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/data/docker/redis5.0/conf/redis.conf",
                "Destination": "/usr/local/etc/redis/redis.conf",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/data/docker/redis5.0/data",
                "Destination": "/data",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
        "Config": {
            "Hostname": "0a1b92c338dc",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "6379/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.10",
                "REDIS_VERSION=5.0.1",
                "REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-5.0.1.tar.gz",
                "REDIS_DOWNLOAD_SHA=82a67c0eec97f9ad379384c30ec391b269e17a3e4596393c808f02db7595abcb"
            ],
            "Cmd": [
                "redis-server",
                "/usr/local/etc/redis/redis.conf"
            ],
            "Image": "docker.io/redis:5.0",
            "Volumes": {
                "/data": {}
            },
            "WorkingDir": "/data",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "d7c289c24b4b15711fabca1cbb5b36a69c10044c0ffeba12e0e2f8d50ed8c9a2",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "6379/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "6379"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/d7c289c24b4b",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "9557808389cc76db0779606589e44d59f33388b02c9e263517f28bfd6a17d770",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.4",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:04",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "158aaf6949aef0aec9bef30e2379966954878dbe97397222327f9bf4ce333d05",
                    "EndpointID": "9557808389cc76db0779606589e44d59f33388b02c9e263517f28bfd6a17d770",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.4",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:04"
                }
            }
        }
    }
]
```

查看 容器 ip 信息：
```
[huangkai@sjq-20 redis5.0]$ docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis5.0
172.17.0.4
```