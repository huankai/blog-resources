---
title: Docker_03_私有仓库搭建
date: {{ date }}
author: huangkai
tags:
    - Docker
---

有了docker hub，为什么还要搭建docker私有仓库？

1、性能考虑：docker hub的访问要通过互联网，性能太低。

2、安全性：更多的时候，镜像不想被外部的人获取，虽然可以在docker hub上申请私有repository，但是需要付费。

# 一、使用 docker registry 安装#
## 1.1、安装环境： ##
ip地址： 192.168.64.150
	
Linux 内核版本：

```
[huangkai@sjq-20 ~]$ uname -a
Linux sjq-20 3.10.0-514.el7.x86_64 #1 SMP Tue Nov 22 16:42:41 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

docker 版本：

```
[huangkai@sjq-20 ~]$ docker -v
Docker version 1.13.1, build 8633870/1.13.1
[huangkai@sjq-20 ~]$ 
```


## 1.2、安装 registry ##

```
[huangkai@sjq-20 ~]$ sudo docker run -d -v /data/docker/registry:/var/lib/registry -p 5000:5000 --restart=always --privileged=true --name registry registry:latest
```

- -v /data/docker/registry:/var/lib/registry 默认情况下，会将仓库存放于容器内的/var/lib/registry目录下，指定本地目录挂载到容器。

- -p 5000:5000 端口映射

- --restart=always 在容器退出时总是重启容器,主要应用在生产环境

- --privileged=true 在CentOS7中的安全模块selinux把权限禁掉了，参数给容器加特权，不加上传镜像会报权限错误OSError: [Errno 13] Permission denied: ‘/tmp/registry/repositories/liibrary’)或者（Received unexpected HTTP status: 500 Internal Server Error）错误

- --name registry 指定容器的名称

通过执行上面的run命令，实际我们已经完成了Docker私有仓库的搭建

查看docker 进程：

```
[huangkai@sjq-20 ~]$ docker ps
CONTAINER ID        IMAGE                    COMMAND                  CREATED             STATUS              PORTS                               NAMES
b63b73507060        registry:latest          "/entrypoint.sh /e..."   39 minutes ago      Up 30 minutes       0.0.0.0:5000->5000/tcp              registry
[huangkai@sjq-20 ~]$ 
```





## 1.3、 推送nginx到私有 Registry ##

下载 nginx：

```
[huangkai@sjq-20 ~]$ docker pull nginx
```

查看镜像：
```
[huangkai@sjq-20 ~]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx             1.14.0              ecc98fc2f376        12 days ago         109 MB
docker.io/registry          latest              2e2f252f3c88        6 weeks ago         33.3 MB
```
使用 tag修改名称

```
[huangkai@sjq-20 ~]$ docker tag docker.io/nginx:1.14.0 192.168.64.150:5000/nginx
```
查看镜像：
```
[huangkai@sjq-20 ~]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx             1.14.0              ecc98fc2f376        12 days ago         109 MB
192.168.64.150:5000/nginx   latest              ecc98fc2f376        12 days ago         109 MB
docker.io/registry          latest              2e2f252f3c88        6 weeks ago         33.3 MB
```

推送到本地私有仓库

先修改docker 配置：

```
[huangkai@sjq-20 registry]$ sudo vim /etc/docker/daemon.json

添加如下内容:

"insecure-registries":["192.168.64.150:5000"]
```

重启docker :

```
[huangkai@sjq-20 registry]$ sudo systemctl restart docker
```

push 到本地私有仓库
```
[huangkai@sjq-20 registry]$ docker push 192.168.64.150:5000/nginx:latest
The push refers to a repository [192.168.64.150:5000/nginx]
19c605f267f4: Pushed 
f4a5f8f59caa: Pushed 
237472299760: Pushed 
latest: digest: sha256:d43aa3719937f9df0502f8258f3034a21b720b5b9bbf01bbfdbd09871aac8930 size: 948
[huangkai@sjq-20 registry]$ 
```
执行完成后，会在 /data/docker/registry 目录下创建相应的目录与文件。

再次查看镜像如下：

```
[huangkai@sjq-20 registry]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
192.168.64.150:5000/nginx   latest              ecc98fc2f376        12 days ago         109 MB
docker.io/nginx             1.14.0              ecc98fc2f376        12 days ago         109 MB
docker.io/registry          latest              2e2f252f3c88        6 weeks ago         33.3 MB
[huangkai@sjq-20 registry]$
```

查询镜像：


使用 docker search 查询会报404
```
[huangkai@sjq-20 nginx]$ docker search 192.168.64.150:5000/ngin
Error response from daemon: Unexpected status code 404
```

可以使用V2版本的api查询
```
[huangkai@sjq-20 registry]$ curl http://192.168.64.150:5000/v2/_catalog
{"repositories":["nginx"]}
[huangkai@sjq-20 registry]$
```

## 1.4、从私有云pull Registry ##

使用另一台机器从私有云下载镜像 ,ip地址： 192.168.64.151

先修改docker 配置：

```
[huangkai@sjq-21 registry]$ sudo vim /etc/docker/daemon.json

添加如下内容:

"insecure-registries":["192.168.64.150:5000"]  #修改为私有云的ip和端口号
```

重启docker :

```
[huangkai@sjq-21 registry]$ sudo systemctl restart docker
```



pull 镜像

```
[huangkai@sjq-21 ~]$ docker pull 192.168.64.150:5000/nginx
Using default tag: latest
Trying to pull repository 192.168.64.150:5000/nginx ... 
sha256:d43aa3719937f9df0502f8258f3034a21b720b5b9bbf01bbfdbd09871aac8930: Pulling from 192.168.64.150:5000/nginx
Digest: sha256:d43aa3719937f9df0502f8258f3034a21b720b5b9bbf01bbfdbd09871aac8930
Status: Downloaded newer image for 192.168.64.150:5000/nginx:latest

```

查看镜像:

```
[huangkai@sjq-21 ~]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
192.168.64.150:5000/nginx   latest              ecc98fc2f376        12 days ago         109 MB
[huangkai@sjq-21 ~]$ 
```

# 二、使用 nexus配置私有仓库 #

先按照上一节的将 nexus3 安装并启动 nexus 容器

## 2.1、创建仓库： ##
可以先创建 Blob Stores 指定目录即可。

使用浏览器访问 nexus，并登陆,创建docker repositories

点击 create repository 后，选择 docker(hosted),可以看到 Docker 有三种类型，分别是 docker(group)，docker(hosted)，docker(proxy)。其含义解释如下：

- hosted : 本地存储，即同 docker 官方仓库一样提供本地私服功能
- proxy : 提供代理其他仓库的类型，如 docker 中央仓库
- group : 组类型，实质作用是组合多个仓库为一个地址

### 2.1.1、创建 hosted仓库 ###
进入创建页面：

如上图：
name :给定一个名称
http：开启http，并设置一个端口号，
Enable Docker V1 API: 如果需要同时支持 V1 版本请勾选此项（不建议勾选）
Hosted -> Deployment pollcy: 请选择 Allow redeploy 否则无法上传 Docker 镜像，默认为 Allow redeploy。

其它保持默认即可。

## 2.2、添加访问权限 ##

菜单 Security->Realms 把 Docker Bearer Token Realm 移到右边的框中保存。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_03.png)

添加用户规则：菜单 Security->Roles->Create role ->Nexus Role 在 Privlleges 选项搜索 docker 把相应的规则移动到右边的框中然后保存。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_04.png)

添加用户：菜单 Security->Users->Create local user 在 Roles 选项中选中刚才创建的规则移动到右边的窗口保存。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_05.png)



### 2.1.1、创建 hosted 仓库 ###

用于存储用户自定义的镜像

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_06_01.png)

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_06_02.png)

### 2.1.2、创建 proxy 仓库 ###

如果用户在 pull镜像时，这个镜像不存在，会到第三方下载镜像并保存在 docker-proxy 类型中，构建第三方镜像，跟maven一个道理

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_07_01.png)

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_07_02.png)

### 2.1.3、创建 group 仓库 ###
用来合并 hosted 和 proxy,只需要暴露这个端口对外 pull，会从 hosted 和 proxy 两种类型中搜索镜像。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_08_01.png)

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_08-02.png)


## 2.3、配置宿主机docker，并认证 ##

sudo vim /etc/docker/daemon.json 在文件中添加如下内容
```
"insecure-registries":["192.168.64.150:8870"]
```

重启docker


```
sudo systemctl restart docker
```

在重启之后，看上面创建docker 创建时， <font color='red'>指定了 8870 端口号(group 类型) 和 8871端口号(hosted类型)，这里也需要将 nexus容器中的8870端口号暴露出来，供所有访问的拉取镜像(pull) ，8871 端口供管理员上传镜像(push)，因为 8870(group 端口号)的服务不会提供具体的存储服务，主要作用是类似于一个反向代理，可以把多个仓库(比如 hosted 私服和 proxy)组合成一个地址提供访问其实也许你应该会想到，为什么 push 需要一个端口、pull又需要另一个端口，为什么不能搞成一个端口，让管理员指定一个默认的存储库呢，对于这个问题，nexus设计就是如此，不排除以后会在新版本中升级，请查看 ：https://issues.sonatype.org/browse/NEXUS-10471 </font> 先停止 nexus 容器
，将重新运行新 nexus 暴露 8870端口号
```
docker run -d -p 8081:8081 -p 8870:8870 -p 8871:8871 --restart=always --name nexus --privileged=true -v /data/docker/nexus:/nexus-data sonatype/nexus3
```


登陆认证：

在通过nexus完成私有镜像仓库的构建后，首先需要进行登录认证才能进行后续的操作，私有镜像仓库登录认证的语法和格式：docker login <nexus-hostname>:<repository-port>。假设上述的nexus部署在IP地址为192.168.64.150主机上，私有镜像的端口为8870，则通过执行如下的命令登录私有镜像仓库：

```
[huangkai@sjq-20 docker]$ docker login 192.168.64.150:8870
Username: docker
Password: 
Login Succeeded
[huangkai@sjq-20 docker]
```
登录时，需要提供用户名和密码。认证的信息会被保存在~/.docker/config.json文件，在后续与私有镜像仓库交互时就可以被重用，而不需要每次都进行登录认证。

## 2.4、推送镜像到nexus ##

要共享一个镜像，可以通过将其发布到托管存储库，然后其它人员就可以通过存储库获取自己需要的镜像。在将镜像推送到存储库之前，需要对镜像进行标记。当标记图像时，可以使用镜像标识符（imageId）或者镜像名称（imageName）。标识镜像的语法和格式：docker tag <imageId or imageName> <nexus-hostname>:<repository-port>/<image>:<tag>。假设这里将 tomcat:8.5 镜像标识为私有镜像仓库(192.168.64.150:8870)中的镜像，标识的执行命令如下：

```
[huangkai@sjq-20 ~]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
docker.io/tomcat            8.5                 ca9e2fccef98        25 hours ago        463 MB
[huangkai@sjq-20 ~]$ docker tag ca9e2fccef98 192.168.64.150:8871/tomcat:8.5
[huangkai@sjq-20 ~]$ docker images  #如下，可以看到自己的镜像创建成功
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
192.168.64.150:8871/tomcat   8.5                 ca9e2fccef98        25 hours ago        463 MB
docker.io/tomcat             8.5                 ca9e2fccef98        25 hours ago        463 MB
[huangkai@sjq-20 ~]$ 
```

一旦镜像标识完成后，就可以通过的docker push命令将镜像推送到私有仓库中。推送镜像到私有镜像仓库的语法和格式为docker push <nexus-hostname>:<repository-port>/<image>:<tag>，通过下面的命令，将上述打完标签的镜像上传至私有镜像仓库：

```
[huangkai@sjq-20 ~]$ docker push 192.168.64.150:8871/tomcat:8.5
The push refers to a repository [192.168.64.150:8871/tomcat]
18bbfbcc62ab: Pushed 
ea8cd57aea48: Pushed 
9883f52bceab: Pushed 
d277f245cc6b: Pushed 
0a743e17e0d0: Pushed 
20468c041c0a: Pushed 
185c62a48a71: Pushed 
d683471ab65a: Pushed 
0f25831f224d: Pushed 
08a01612ffca: Pushed 
8bb25f9cdc41: Pushed 
f715ed19c28b: Pushed 
8.5: digest: sha256:119c1d475e72254afe4359a2e0a0daf1b5909835249172645b70acf575e2818a size: 2836
[huangkai@sjq-20 ~]$
```
如下图所示，tomcat 已被上传到本地私服中。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_09.png)


## 2.5、使用另一台服务器拉取镜像 ##

修改配置如下：

```
[huangkai@sjq-21 ~]$ vim /etc/docker/daemon.json
"insecure-registries":["192.168.64.150:8870"]
```
重启 docker:

```
[huangkai@sjq-21 ~]$ systemctl restart docker
```

拉取的语法和格式：docker pull <nexus-hostname>:<repository-port>/<image>:<tag>。假设从本文构建的私有镜像仓库中拉取tomcat:8.5，执行命令如下所示：

```

[huangkai@sjq-21 ~]$ docker login 192.168.64.150:8870 #使用另一台服务器(192.168.64.151)，先登陆 docker repository
Username: docker
Password: 
Login Succeeded
[huangkai@sjq-21 ~]$ docker pull 192.168.64.150:8870/tomcat:8.5 #如果nexus中已存在该镜像，直接下载，如果存在，会在 proxy类型的仓库中下载到Nexus 再下载到用户服务器本地镜像中。
Trying to pull repository 192.168.64.150:8870/tomcat ... 
sha256:119c1d475e72254afe4359a2e0a0daf1b5909835249172645b70acf575e2818a: Pulling from 192.168.64.150:8870/tomcat
bc9ab73e5b14: Pull complete 
193a6306c92a: Pull complete 
e5c3f8c317dc: Pull complete 
d21441932c53: Pull complete 
fa76b0d25092: Pull complete 
346fd8610875: Pull complete 
3ca5d6af9022: Pull complete 
c06cfa2cea32: Pull complete 
205950a5a114: Pull complete 
6332a55c669e: Pull complete 
b5efe96df0e8: Pull complete 
b4e0e542b56a: Pull complete 
Digest: sha256:119c1d475e72254afe4359a2e0a0daf1b5909835249172645b70acf575e2818a
Status: Downloaded newer image for 192.168.64.150:8870/tomcat:8.5
[huangkai@sjq-21 ~]$ docker images #查看镜像，tomcat已下载
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
192.168.64.150:8870/tomcat   8.5                 ca9e2fccef98        26 hours ago        463 MB
[huangkai@sjq-21 ~]$
```

