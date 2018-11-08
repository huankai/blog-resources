---
title: Docker_02_安装 nexus
date: {{ date }}
author: huangkai
tags:
    - Docker
---

nexus3 使用的数据存储的 OrientDB 数据库。


# 下载镜像 #

```
[huangkai@sjq-20 log]$ docker pull sonatype/nexus3
```
# 查看镜像： #

```
[huangkai@sjq-20 log]$ docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
docker.io/sonatype/nexus3   latest              f2014d39f023        2 weeks ago         509 MB
[huangkai@sjq-20 log]$ 
```

# 启动镜像： #

首先，创建一个目录，用于为Nexus3提供持久化存储，nexus会使用 200 做为 UID，所以给目录下的所有子级都授权。

```
[huangkai@sjq-20 ~]$   mkdir -p /data/docker/nexus && chown -R 200 /data/docker/nexus

```
启动nexus:

```

[huangkai@sjq-20 ~]$  docker run -d -p 8081:8081 --name nexus --restart=always --privileged=true -v /data/docker/nexus:/nexus-data sonatype/nexus3
```

上面的 `-v /data/docker/nexus:/nexus-data` 是将宿主机的`/data/docker/nexus`目录与 docker 容器中的`/nexus-data`目录映射，保证数据的安全性。

可以添加 NEXUS_CONTEXT=nexus 指定上下文路径

注意：nexus3 存储的磁盘最小需要 4G，可以通过` -e INSTALL4J_ADD_VM_PARAMS="-XX:MaxDirectMemorySize=3g" `参数指定。

# 查看日志： #

```
[huangkai@sjq-20 ~]$  docker logs -f nexus
```

如果日志没有出错，可以使用浏览器访问 : http：//ip:8081，如下 ：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_nexus_01.png)



密码的登陆账号 ：admin ，密码: admin123

文档地址：https://hub.docker.com/r/sonatype/nexus3/

maven 的 仓库(Repositories) 默认会有 4 个(snapshots/releases/public/central)，如下图：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_nexus_02.png)

- maven-snapshots: 仓库类型为 hosted,为本地 snapshots 版本；
- maven-releases: 仓库类型为 hosted,为本地 releases 版本；
- maven-central:仓库类型为 proxy，可代理第三方仓库，可点击进去后修改 proxy->remote storage 为阿里镜像地址。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_nexus_03.png)

- maven-public : 仓库类型为 group，是上面三者的合并，可点击进去后查看 Group -> Member repositories 右边的 Members，需要注意顺序。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/docker/docker_nexus_04.png)

#  #maven 配置使用nexus :

在 ${MAVEN_HOME}/conf/setting.xml 文件内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>E:\repository</localRepository>
  <pluginGroups></pluginGroups>
  <proxies></proxies>
  <servers>
  <server>
      <id>nexus</id> <!-- 值要和 <mirror><id>值一样 -->
      <username>admin</username>
      <password>admin123</password>
    </server>
  </servers>
  <mirrors>
	<mirror>
		<id>nexus</id>
		<mirrorOf>*</mirrorOf>
		<url>http://192.168.64.150:8081/repository/maven-public/</url>
	</mirror>
  </mirrors>
  <profiles>
	<profile>
		<id>development</id>
		<activation>
			<jdk>1.8</jdk>
			<activeByDefault>true</activeByDefault>
		</activation>
     <properties>
          <maven.compiler.source>1.8</maven.compiler.source>
          <maven.compiler.target>1.8</maven.compiler.target>
          <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
     </properties>
	</profile>
	<profile>
      <id>nexus</id>
	  <activation>
			<activeByDefault>true</activeByDefault>
		</activation>
      <repositories>
        <repository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </repository>
      </repositories>
     <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>http://central</url>
          <releases><enabled>true</enabled></releases>
          <snapshots><enabled>true</enabled></snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>    
  </profiles>
</settings>

```

maven 项目pom.xml文件内容：

```
...
<!-- 发布到私服 -->
<distributionManagement>
    <repository>
        <id>nexus</id> <!-- 注意id 值要和 settings.xml文件中的  <server><id> 值一样
        <name>Releases</name>
        <url>http://192.168.64.150:8081/repository/maven-releases</url>
    </repository>
    <snapshotRepository>
        <id>nexus</id>
        <name>Snapshot</name>
        <url>http://192.168.64.150:8081/repository/maven-snapshots</url>
    </snapshotRepository>
</distributionManagement>
..
```

