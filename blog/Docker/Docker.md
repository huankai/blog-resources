---
title: Docker
date: {{ date }}
author: huangkai
tags:
    - Docker
---

# 一、介绍 #
## 1.1、镜像 ##
 Docker镜像(Image)就是一个只读的模板，可以用来创建Docker容器，一个镜像可以启动多个Docker容器。
## 1.2、容器 ##
Docker利用容器(Container)来运行应用，容器是从镜像创建的运行实利，它可以启动、停止、删除等。每个容器都是相互隔离的、保证安全的平台。
可以把容器看成是一个简单的Linux环境（包括ROOT用户权限、进程空间、用户空间、网络等）和运行在其中的应用程序。

## 1.3、 Registry ##
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
[root@sjq01 ~]$ yum -y install docker
```

## 2.2、启动 ： ##
```
[root@sjq01 ~]$ systemctl start docker.service
```

## 2.4、停止 ： ##
```
[root@sjq01 ~]$ systemctl stop docker.service
```

## 2.4、配置国内镜像： ##
Docker默认的镜像为 https://hub.docker.com/ ，从此镜像下载会非常慢，可根据如下配置使用国内镜像下载
```
[root@sjq01 ~]$ vim /etc/docker/daemon.json
# 添加内容如下：

{
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}

```
重启Docker:
配置完之后执行下面的命令，以使docker的配置文件生效
```
[root@sjq01 ~]$systemctl restart docker
```

# 三、常用操作 #

## 3.1、列出镜像 ##
使用 `docker images` 命令列出 镜像，如下，表示有 nexus镜像
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

# 四、Docker容器操作  #

## 4.1、启动容器 ##
- 以交互式启动：
docker run -it --name 容器名称 镜像 /bin/bash
```
[root@sjq01 ~]# docker run -it --name my-nexus docker.io/sonatype/nexus:latest /bin/bash
```
容器名称可以任意指定，必须唯一
退出交互模式，直接输入 `exit` 退出

- 以守护进程方式启动：
docker run -d --name 容器名称 镜像
```
[root@sjq01 ~]# docker run -d --name my-nexus docker.io/sonatype/nexus:latest
```
容器名称必须唯一

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
