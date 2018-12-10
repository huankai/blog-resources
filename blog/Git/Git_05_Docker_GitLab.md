---
title: 使用 docker 搭建GitLab及配置
date: {{ date }}
author: huangkai
tags:
    - Git
---

## 创建目录,配置 docker-compose: ##

```
[huangkai@sjq150 ~]$ sudo mkdir -p /data/docker

[huangkai@sjq150 ~]$ cd /data/docker
[huangkai@sjq150 ~]$ vim docker-compose.yml
```

## `docker-compose` 文件如下： ##

```

services: 
	gitlab: 
	    image: gitlab/gitlab-ce
	    container_name: gitlab
	    # 指定 hostname，如果不定义，在git clone 地址不会使用此 ip 而生成一个不规则的字符串
	    hostname: 192.168.1.150
	    ports: 
			# http 端口
	      - "8085:80"
	      - # ssh 端口
	      - "88:22"
	    volumes:
	      - "/etc/timezone:/etc/timezone:ro"
	      - "/etc/localtime:/etc/localtime:ro"
	      - "/data/docker/gitlab/config:/etc/gitlab"
	      - "/data/docker/gitlab/logs:/var/log/gitlab"
	      - "/data/docker/gitlab/data:/var/opt/gitlab"
	    restart: "always"
```

## 修改 gitlab配置： ##

```
[huangkai@sjq150 ~]$ mkdir -p /data/docker/gitlab/config
[huangkai@sjq150 ~]$ cd /data/docker/gitlab/config
[huangkai@sjq150 config]$ vim gitlab.rb
```

gitlab.rb 内容请 [点击这里](https://github.com/huankai/hk-config/blob/master/doc/docker/gitlab/config/gitlab.rb) 查看

## 说明 ##
gitlab 内置的数据库为 postgresql，缓存为 redis ，使用 nginx 作为http 服务器，如果你不想使用内置的这些应用，想使用 `docker-compose` 自定义编排， 请 [点击这里](https://github.com/huankai/hk-config/blob/master/doc/docker/docker-compose.yml) 查看。