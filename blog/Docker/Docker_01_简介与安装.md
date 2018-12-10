---
title: Docker_01_简介与安装
date: {{ date }}
author: huangkai
tags:
    - Docker
---

# 一、介绍 #
## 1.1、Docker 介绍： ##
Docker 是一个开源的应用容器引擎，基于 Go 语言并遵从Apache2.0协议开源，可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、bare metal、OpenStack 集群和其他的基础应用平台。 
Docker 容器是完全使用沙箱机制，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

Docker通常用于如下场景：
- web应用的自动化打包和发布
- 自动化测试和持续集成、发布
- 在服务型环境中部署和调整数据库或其他的后台应用
- 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。

Docker的优点：
- 简化程序：
Docker 让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，便可以实现虚拟化。Docker改变了虚拟化的方式，使开发者可以直接将自己的成果放入Docker中进行管理。方便快捷已经是 Docker的最大优势，过去需要用数天乃至数周的任务，在Docker容器的处理下，只需要数秒就能完成。
- 避免选择恐惧症
如果你有选择恐惧症，还是资深患者。Docker 帮你打包你的纠结！比如 Docker 镜像；Docker 镜像中包含了运行环境和配置，所以 Docker 可以简化部署多种应用实例工作。比如 Web 应用、后台应用、数据库应用、大数据应用比如 Hadoop 集群、消息队列等等都可以打包成一个镜像部署。
- 节省开支
一方面，云计算时代到来，使开发者不必为了追求效果而配置高额的硬件，Docker 改变了高性能必然高价格的思维定势。Docker 与云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题，也改变了虚拟化的方式。

## 1.2、Docker 基本概念： ##
### 1.2.1、镜像 ###
 Docker镜像(Image)就是一个只读的模板，可以用来创建Docker容器，一个镜像可以启动多个Docker容器。
### 1.2.2、容器 ###
Docker利用容器(Container)来运行应用，容器是从镜像创建的运行实利，它可以启动、停止、删除等。每个容器都是相互隔离的、保证安全的平台。
可以把容器看成是一个简单的Linux环境（包括ROOT用户权限、进程空间、用户空间、网络等）和运行在其中的应用程序。

### 1.2.3、 Registry ###
Registry(仓库)是集中存放镜像文件的场所。
Registry对其中的镜像进行分类管理。
- 一个Registry 会有多个Repository
- 一个Registry会有多个不同的tag与Image

比如名称为 centos 的repository 下，有tag 为 6 和 7 的镜像。

Registry 分为公有与私有的两种形式：
- 最大的公有Registry是Docker Hub,存放了大量的镜像供用户下载
- 国内的公有Registry有USTC、网易云、aliCloud、DaoCloud等，可供大陆用户更加稳定的访问
- 用户可以在本地创建一个私有Registry

用户创建了自己的镜像之后就可以使用push 命令上传到公有Registry或私有Registry中，这样下次在另一台机器使用这个镜像时就只需要从Registry pull下来就可以运行了。
 
# 二、安装 #
##  2.1、安装环境： ##
操作系统：CentOS Linux release 7.3.1611 (Core)
Docker版本：Docker version 1.12.6, build 3e8e77d/1.12.6

查看是否有安装Docker
```
[root@sjq01 ~]# yum list installed|grep docker
docker.x86_64                     2:1.12.6-71.git3e8e77d.el7.centos.1  @extras  
docker-client.x86_64              2:1.12.6-71.git3e8e77d.el7.centos.1  @extras  
docker-common.x86_64              2:1.12.6-71.git3e8e77d.el7.centos.1  @extras  
[root@sjq01 ~]# 
```
如果上面命令执行有信息出来，执行 `yum -y remove xxx` 删除，xxx指定的显示的信息：
```
[root@sjq01 ~]# yum -y remove docker.x86_64
[root@sjq01 ~]# yum -y remove docker-client.x86_64
[root@sjq01 ~]# yum -y remove docker-common.x86_64 
```
- yum 安装

```
# 安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
[root@sjq01 ~]# yum install -y yum-utils device-mapper-persistent-data lvm2

# 设置yum源
[root@sjq01 ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 可以查看所有仓库中所有docker版本，并选择特定版本安装
[root@sjq01 ~]# yum list docker-ce --showduplicates | sort -r
 * updates: mirrors.163.com
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror
Installed Packages
 * extras: mirrors.aliyun.com
 * epel: mirrors.aliyun.com
docker-ce.x86_64            3:18.09.0-3.el7                    docker-ce-stable 
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.06.1.ce-3.el7                   @docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable 
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.09.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.06.0.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.3.ce-1.el7                   docker-ce-stable 
docker-ce.x86_64            17.03.2.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable 
docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable 
 * base: mirrors.aliyun.com
Available Packages

 
[root@sjq01 ~]# sudo yum install docker-ce -y #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版 18.06.1
[root@sjq01 ~]# yum -y install docker-ce-18.06.1.ce
```

验证：

```
[root@sjq01 ~]# docker version
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.3
 Git commit:        e68fc7a
 Built:             Tue Aug 21 17:23:03 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.3
  Git commit:       e68fc7a
  Built:            Tue Aug 21 17:25:29 2018
  OS/Arch:          linux/amd64
  Experimental:     false
[root@sjq150 ~]#

```
## 2.2、启动 ： ##

```
[root@sjq01 ~]# systemctl start docker.service
```

常见错误：

1、SELinux is not supported with the overlay2 graph driver on this kernel

```
[root@sjq-01 ~]# systemctl start docker.service 
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.
[root@sjq-01 ~]# systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Mon 2018-03-12 17:21:57 CST; 3s ago
     Docs: http://docs.docker.com
  Process: 3623 ExecStart=/usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userland-proxy-path=/usr/libexec/docker/docker-proxy-current --seccomp-profile=/etc/docker/seccomp.json $OPTIONS $DOCKER_STORAGE_OPTIONS $DOCKER_NETWORK_OPTIONS $ADD_REGISTRY $BLOCK_REGISTRY $INSECURE_REGISTRY $REGISTRIES (code=exited, status=1/FAILURE)
 Main PID: 3623 (code=exited, status=1/FAILURE)

Mar 12 17:21:56 sjq-01 systemd[1]: Starting Docker Application Container Engine...
Mar 12 17:21:56 sjq-01 dockerd-current[3623]: time="2018-03-12T17:21:56.051545029+08:00" level=info msg="libcontainerd: new containerd process, pid: 3628"
Mar 12 17:21:57 sjq-01 dockerd-current[3623]: Error starting daemon: SELinux is not supported with the overlay2 graph driver on this kernel. Either boot into a newer kernel or disable selinux in docker (--selinux-enabled=false)
Mar 12 17:21:57 sjq-01 systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
Mar 12 17:21:57 sjq-01 systemd[1]: Failed to start Docker Application Container Engine.
Mar 12 17:21:57 sjq-01 systemd[1]: Unit docker.service entered failed state.
Mar 12 17:21:57 sjq-01 systemd[1]: docker.service failed.
```

解决方法：

```
vim  /etc/sysconfig/docker
添加配置内容如下：
	DOCKER_OPTS="--storage-driver=devicemapper"
并注释配置内容：
	# OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'

```
详见：https://docs.docker.com/storage/storagedriver/select-storage-driver/

2、运行容器时出错：

```
[root@sjq01 ~]# docker run hello-world
docker: Error response from daemon: OCI runtime create failed: unable to retrieve OCI runtime error (open /run/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/225edd3d808116d3cc5992849e60bf5369ace67c291a066ebae4ca5784bcce7a/log.json: no such file or directory): docker-runc did not terminate sucessfully: unknown.
[root@sjq01 ~]# uname -rs  #查看系统内核版本
Linux 3.10.0-327.el7.x86_64
[root@sjq01 ~]# yum install http://mirror.centos.org/centos/7/os/x86_64/Packages/libseccomp-2.3.1-3.el7.x86_64.rpm
```

## 2.4、停止 ： ##
```
[root@sjq01 ~]# systemctl stop docker.service
```

## 2.5、配置国内镜像： ##
Docker默认的镜像为 https://hub.docker.com/ ，从此镜像下载会非常慢，可根据如下配置使用国内镜像下载
```
[root@sjq01 ~]$ vim /etc/docker/daemon.json
# 添加内容如下：

{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}

或者

{
"registry-mirrors": ["https://registry.docker-cn.com"]
}
```
重启Docker:
配置完之后执行下面的命令，以使docker的配置文件生效
```
[root@sjq01 ~]$systemctl restart docker
```

## 2.6、普通用户操作docker： ##

```
[root@sjq01 ~]# groupadd docker  #添加docker组
[root@sjq01 ~]# gpasswd -a huangkai docker #将用户huangkai添加到docker组中
Adding user huangkai to group docker  
[root@sjq01 ~]# systemctl restart docker #重启 docker
[root@sjq01 ~]# sudo chmod a+rw /var/run/docker.sock  #修改docker.sock文件的权限
```
## 2.7、开机启动docker： ##

```
[root@sjq01 ~]# systemctl enable docker
```

# 三、常用操作 #

## 3.1、列出镜像 ##
使用 `docker images` 命令列出已存在的镜像，如下，表示有 nexus镜像
```
[root@sjq01 ~]# docker images
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
docker.io/sonatype/nexus   latest              f7d8039f8626        2 weeks ago         454.6 MB
```

## 3.2、删除镜像 ##
docker rmi (名称:tag) 或 docker rmi (image id)
名称与tag中间用英文冒号分隔，可以确定一个镜像，也可以直接使用 image id
```
[root@sjq01 ~]# docker rmi docker.io/sonatype/nexus:latest
```
或使用镜像id删除
```
[root@sjq01 ~]# docker rmi f7d8039f8626
```

## 3.3、导入/导出镜像 ##
- 导入
将本机文件导入到docker中
执行命令的语法：
 <font color='red'>docker load < 完整的文件路径名</font>
```
[root@sjq01 ~]# docker load < /root/nexus.tar.gz
```

- 导出
 将Docker镜像导出到本机文件中
执行命令的语法：
<font color='red'>docker save (名称:tag) > /root/文件名.tar.gz</font>
会将指定的镜像导出到/root目录下，如导出nexus 到本地文件
```
[root@sjq01 ~]# docker save docker.io/sonatype/nexus:latest > /root/nexus.tar.gz
```

## 3.4、查看容器信息 ##

`docker inspect 容器Id`

# 四、Docker容器操作  #

## 4.1、启动容器 ##
- 以交互式启动：
`docker run -it --name 容器名称 镜像 /bin/bash`
`-i` : 以交互方式运行容器
`-t` ：表示告诉docker为要创建的容器分配一个ty伪终端
`--name`: 指定创建的容器名，如果无此参数，docker将生成随机的容器名，容器名称可以任意指定，必须唯一
`镜像`:要运行的镜像
`/bin/bash` : 以交互模式启动指定的 shell
```
[root@sjq01 ~]# docker run -it --name my-nexus docker.io/sonatype/nexus:latest /bin/bash
```
退出交互模式，直接输入 `exit` 退出

- 以守护进程方式启动：
`docker run -d --name 容器名称 镜像`
`-d`:守护进程方式启动
```
[root@sjq01 ~]# docker run -d --name my-nexus docker.io/sonatype/nexus:latest
```
容器名称必须唯一

上面的启动方式是启动一个不存在的容器，如果这个服务已创建，则不需要再次以上面的方式启动，可以直接使用 <font color='red'>**docker start 容器名**</font>启动，如：
```
docker start my-nexus
```
重启：
```
docker restart my-nexus
```

## 4.2、停止容器 ##
docker stop 容器名称或容器Id
```
[root@sjq01 ~]# docker stop my-nexus
```

## 4.3、删除容器 ##
删除指定容器 ：`docker rm 容器名称或容器Id`
删除所有容器： `docker rm 'docker ps -a -q'`

## 4.4、查看容器 ##
使用 ``docker ps -a`` 查看：
```
[root@sjq01 ~]# docker ps -a
CONTAINER ID        IMAGE                             COMMAND                  CREATED             STATUS              PORTS                    NAMES
d5579464bcef        f7d8039f8626                      "/bin/sh -c '${JAVA_H"   12 minutes ago      Up 12 minutes       0.0.0.0:8081->8081/tcp   nexus
2b2ca50cb7f8        docker.io/sonatype/nexus:latest   "-p 8081:8081 --name "   15 hours ago        Created             8081/tcp                 stupefied_borg
cb71f72f94b9        docker.io/sonatype/nexus:latest   "-p 8081:8081 --name"    15 hours ago        Created             8081/tcp                 prickly_lichterman
[root@sjq01 ~]# 
```

# 五、Docker 搭建Tomcat服务 # 
使用``docker pull tomcat`` 下载镜像

启动Docker Tomcat服务 ：
``docker run -d --name my-tomcat -p 8888:8080 镜像``
这里的`-p 8888:8080` 指的是宿主机的8888端口号映射到docker tomcat容器的8080端口号，此时如果启动成功，访问宿主机的8888服务，其实就是访问 docker tomcat的8080服务。

进入到 Docker Tomcat运行环境：
``docker exec -it my-tomcat /bin/bash``

将 war 应用上传到 Docker Tomcat webapps目录中：
``docker cp 应用程序war包 my-tomcat:/usr/local/tomcat/webapps``

回车上传完成后，tomcat 会自动加载上传的war，不需要手动重启。


# 六、Docker 搭建 Mysql 服务  #

## 6.1、搭建mysql 5.7版本 ##

使用``docker pull mysql:5.7`` 下载MySql镜像
查看docker 镜像：
```
[root@sjq-01 mysql]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/mysql     5.7                 5d4d51c57ea8        3 weeks ago         374 MB
[root@sjq-01 mysql]# 
```
创建mysql相关目录 ：
```
[root@sjq-01 mysql]# mkdir -p /data/docker/mysql/conf # mysql配置目录
[root@sjq-01 mysql]# mkdir -p /data/docker/mysql/logs  #mysql log 目录 
[root@sjq-01 mysql]# mkdir -p /data/docker/mysql/data # mysql data目录
``` 

创建配置文件：
```
[root@sjq-01 mysql]# cd /data/docker/mysql/conf
[root@sjq-01 conf]# vim my.cnf
添加如下内容：

[mysqld]
character-set-server=utf8 #编码设置
lower_case_table_names=1  #忽略表名大小写
```



启动 Docker Mysql 服务
```
[root@sjq-01 mysql]# docker run --name mysql --restart=always -v /data/docker/mysql/data/:/var/lib/mysql -v /data/docker/mysql/conf/my.cnf:/etc/mysql/conf.d/my.cnf -v /data/docker/mysql/logs:/logs -d -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 docker.io/mysql:5.7
546176b9d5915804e85dde6ba0a59f248691078390fed0750751cc464122ba15
[root@sjq-01 mysql]#
```
上面启动参数解释：
-v /data/docker/mysql/data/:/var/lib/mysql # 将主机/data/mysql/data/挂载到容器的/var/lib/mysql
-v /data/docker/mysql/conf/:/etc/mysql/conf.d  #将主机/data/mysql/conf/挂载到容器的/etc/mysql/conf.d
-v /data/docker/mysql/logs:/logs		#将主机/data/mysql/logs目录挂载到容器的/logs
-e MYSQL_ROOT_PASSWORD=root 设置环境变量，mysql root账号的密码

其中 -v 参数可以指定多个，-e 参数也可以指定多个，查看更多环境变量请点击 [这里](https://hub.docker.com/_/mysql/) 查看

查看 docker 进程如下，可知， mysql服务已启动成功
```
[root@sjq-01 mysql]# docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
546176b9d591        docker.io/mysql:5.7   "docker-entrypoint..."   10 minutes ago      Up 10 minutes       0.0.0.0:3306->3306/tcp   mysql
[root@sjq-01 mysql]# 
```

MySQL(5.7.19)的默认配置文件是 /etc/mysql/my.cnf 文件。如果想要自定义配置，建议在 /etc/mysql/conf.d 目录中创建 .cnf 文件。新建的文件可以任意起名，只要保证后缀名是 cnf 即可。新建的文件中的配置项可以覆盖 /etc/mysql/my.cnf 中的配置项。

进入docker mysql 容器
```
[root@sjq-01 mysql]# docker exec -it mysql /bin/bash
```

登陆mysql服务器：
```
root@546176b9d591:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.21 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
输入运行容器指定的 MYSQL_ROOT_PASSWORD 的值，登陆mysql


## 6.2、搭建mysql 8.0.13 版本 ##

使用``docker pull mysql:8.0.13`` 下载MySql镜像
查看docker 镜像：
```
[huangkai@sjq-20 mysql8.0.13]$ docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
192.168.64.150:8870/tomcat   8.5                 ca9e2fccef98        9 days ago          463 MB
docker.io/tomcat             8.5                 ca9e2fccef98        9 days ago          463 MB
192.168.64.150:8870/maven    3.5-alpine          fb4bb0d89941        2 weeks ago         119 MB
docker.io/mysql              8.0.13              2dd01afbe8df        2 weeks ago         485 MB
192.168.64.150:8870/nginx    latest              dbfc48660aeb        3 weeks ago         109 MB
192.168.64.150:8870/redis    4.0.11              f1897cdc2c6b        3 weeks ago         83.4 MB
docker.io/sonatype/nexus3    latest              f2014d39f023        3 weeks ago         509 MB
docker.io/jenkins            latest              cd14cecfdb3a        3 months ago        696 MB
[huangkai@sjq-20 mysql8.0.13]$
```
创建mysql相关目录 ：
```
[root@sjq-01 mysql8.0.13]# mkdir -p /data/docker/mysql8.0.13/conf # mysql配置目录
[root@sjq-01 mysql8.0.13]# mkdir -p /data/docker/mysql8.0.13/logs  #mysql log 目录 
[root@sjq-01 mysql8.0.13]# mkdir -p /data/docker/mysql8.0.13/data # mysql data目录
``` 

创建配置文件：

```
[root@sjq-01 mysql8.0.13]# cd /data/docker/mysql8.0.13/conf
[root@sjq-01 conf]# vim my.cnf
```

添加如下内容：
```
[msqld]

# 端口号配置，默认为 3306
# port=3306

#忽略表名大小写
lower_case_table_names=1

# 设置 mysql 数据存放目录
datadir=/data/docker/mysql8.0.13/data

#允许最大连接数，默认为 151
max_connections=500

#允许连接失败的次数，可以防止有人从该主机试图攻击数据库系统，默认值为100
max_connect_errors=10

################################  InnoDB         ########################################

# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB

innodb_buffer_pool_size=2G

#innodb_additional_pool_size=20M

innodb_log_file_size=256M

innodb_log_buffer_size=12M

innodb_flush_log_at_trx_commit=2

#innodb_flush_method

#thread_cache=8

#innodb_autoextend_increment=128M

#这里确认是否起用压缩存储功能
innodb_file_per_table=1

#innodb_file_format=barracuda #mysql 8 不支持该功能

#决定压缩程度的参数，如果你设置比较大，那么压缩比较多，耗费的CPU资源也较多；
#相反，如果设置较小的值，那么CPU占用少。默认值6，可以设置0-9#
innodb_compression_level=6

#指定在每个压缩页面可以作为空闲空间的最大比例，
#该参数仅仅应用在设置了innodb_compression_failure_threshold_pct不为零情况下，并且压缩失败率通过了中断点。
#默认值50，可以设置范围是0到75
innodb_compression_pad_pct_max=50

[client]

# 设置mysql客户端连接服务端时默认使用的端口
# port=3306

# client 编码
default-character-set=utf8
```
其他配置参数请参考: https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html


启动 Docker Mysql 服务
```
[root@sjq-20 mysql8.0.13]# docker run --name mysql --restart=always -v /data/docker/mysql8.0.13/data/:/var/lib/mysql -v /data/docker/mysql8.0.13/conf/my.cnf:/etc/mysql/conf.d/my.cnf -v /data/docker/mysql8.0.13/logs:/logs -d -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 docker.io/mysql:8.0.13
546176b9d9915804e85dde6ba0a54f248691078396fed075k751cc464122ba39
[root@sjq-20 mysql]#
```

查看 docker 进程如下，可知， mysql服务已启动成功
```
[huangkai@sjq-20 mysql8.0.13]$ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                                                      NAMES
36b177dacf03        docker.io/mysql:8.0.13   "docker-entrypoint..."   22 minutes ago      Up 6 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp                          mysql8
[huangkai@sjq-20 mysql8.0.13]$ 
```

进入docker mysql 容器
```
[root@sjq-01 mysql]# docker exec -it mysql8 /bin/bash
```

登陆mysql服务器：
```
root@36b177dacf03:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.13 MySQL Community Server - GPL

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
输入运行容器指定的 MYSQL_ROOT_PASSWORD 的值，登陆mysql

