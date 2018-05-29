---
title: GitLab 搭建及配置
date: {{ date }}
author: huangkai
tags:
    - Git
---

# 一、GitLab 简介 #

GitLab 是一个利用Ruby on Rails 开发的开源版本控制系统，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。

它拥有与GitHub类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序（Wall）进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

开源中国代码托管平台 码云 就是基于GitLab项目搭建。

GitLab 分为 GitLab Community Edition(CE) 社区版 和 GitLab Enterprise Edition(EE) 专业版。社区版免费，专业版收费，两个版本在功能上的差异对比，可以参考官方[对比说明](https://about.gitlab.com/features/#compare)

# 二、GitLab 安装和配置 #
安装社区版，GitLab CE 版本：10.8.1这此省略

## 2.1、GitLab安装 ##

**安装要求**
- 操作系统：windows 暂不支持，不受支持的 Unix衍生版有：Arch Linux、Fedora、FreeBSD、Gentoo、macOS
- 内存: 必须不少于 4GB，否则在安装过程中会出现不可预知的问题，这也是官网的推荐。
- Ruby版本：2.3以上。
- CPU： 2核心，可支持500用户，这也是官网的推荐
- 数据库： PostgreSQL (官网推荐) 、MySQL/MariaDB (有些功能会受限)
- 更多要求，请查看[官网文档](https://docs.gitlab.com.cn/ce/install/requirements.html)


通过GitLab官方提供的Omnibus安装包来安装，相对方便。Omnibus安装包套件整合了大部分的套件（Nginx、ruby on rails、git、redis、postgresql等），再不用额外安装这些软件，减轻了绝大部分安装量。

GitLab官方安装文档 ：[CentOS7.x系统](https://www.gitlab.com.cn/installation/#centos-7)

**安装依赖包，并配置postfix服务为GitLab邮件服务**
```
[root@sjq-09 ~]# yum install curl openssh-server openssh-clients postfix cronie
[root@sjq-09 ~]# systemctl start postfix	
[root@sjq-09 ~]# systemctl enable postfix  # 配置开机启动
# 关闭防火墙
[root@sjq-09 ~]# systemctl stop firewalld
```

**两种安装源：**

- 从官方镜像源安装
添加GitLab仓库并安装到服务器上
	
	```
	[root@sjq-09 ~]# curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
	
	[root@sjq-09 ~]# yum install gitlab-ce    # 自动安装最新版本
	```
- 从第三方镜像源安装
官方镜像源在国外，国外安装会很慢，甚至有时因网络问题会无法安装。
国内推荐使用 [清华大学开源软件镜像源](https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/)。
使用 `vim /etc/yum.repos.d/gitlab-ce.repo`创建文件，内容如下：
```
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```
保存、退出vim 后，再执行：
```
[root@sjq-09 ~]# yum makecache   # 更新本地YUM缓存
[root@sjq-09 ~]# yum install gitlab-ce    # 自动安装最新版本
```
修改配置文件`vim /etc/gitlab/gitlab.rb`，绑定域名
```
external_url 'http://xxx.com' #也可以写ip地址
```
启动GitLab，使得配置生效
```
[root@sjq-09 ~]# gitlab-ctl reconfigure
```

**使用浏览器访问GitLab**
首次访问GitLab,系统会让你重新设置管理员的密码,设置成功后会返回登录界面。
默认的管理员账号是root,如果你想更改默认管理员账号,请输入上面设置的新密码登录系统后修改帐号名.

**GitLab相关目录及配置**

```
主配置文件: /etc/gitlab/gitlab.rb
GitLab 文档根目录: /opt/gitlab
默认存储库位置: /var/opt/gitlab/git-data/repositories
GitLab Nginx 配置文件路径:  /var/opt/gitlab/nginx/conf/gitlab-http.conf
Postgresql 数据目录: /var/opt/gitlab/postgresql/data
```

**GitLab由以下服务构成**
- nginx: 静态web服务器
- gitlab-shell: 用于处理Git命令和修改authorized keys列表
- gitlab-workhorse: 轻量级的反向代理服务器
- logrotate：日志文件管理工具
- postgresql：数据库、
- redis：缓存数据库
- sidekiq：用于在后台执行队列任务（异步执行）
- unicorn：An HTTP server for Rack applications，GitLab Rails应用是托管在这个服务器上面的。

## 2.2、配置： ##

**2.2.1、配置SMTP服务**
如果你不想用服务器自带的postfix服务来发邮件，可以改用SMTP服务。
修改GitLab邮件服务配置(gitlab.rb文件)，使用腾讯企业邮箱的SMTP服务器，填写账号和密码：
`vim /etc/gitlab/gitlab.rb`
```
gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "xxx"
gitlab_rails['smtp_password'] = "xxx"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = 'plain'
gitlab_rails['smtp_enable_starttls_auto'] = true
```
使配置生效:
```
[root@sjq-09 ~]# gitlab-ctl reconfigure
[root@sjq-09 ~]# gitlab-rake cache:clear RAILS_ENV=production      # 清除缓存
```

**2.2.2、GitLab配置HTTPS**
GitLab默认是使用HTTP的，可以手动配置为HTTPS
上传SSL证书，创建 SSL目录，用于存放SSL证书
```
[root@sjq-09 ~]# mkdir -p /etc/gitlab/ssl
[root@sjq-09 ~]# chmod 0700 /etc/gitlab/ssl
```
上传证书并修改证书权限
```
[root@sjq-09 ~]# chmod 600 /etc/gitlab/ssl/*

```

修改GitLab的主配置文件
`vim /etc/gitlab/gitlab.rb`

```
external_url "https://xxx.com"
nginx['redirect_http_to_https'] = true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/xxx.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/xxx.com.key"
```

重建配置，使其生效
```
[root@sjq-09 ~]# gitlab-ctl reconfigure

```

以上操作后，GitLab自带的Nginx服务的配置文件 /var/opt/gitlab/nginx/conf/gitlab-http.conf 会被重新修改：

```
server {
  listen *:80;
  server_name xxx.com;
  server_tokens off; ## Don't show the nginx version number, a security best practice
  return 301 https://xxx.com:443$request_uri;
  access_log  /var/log/gitlab/nginx/gitlab_access.log gitlab_access;
  error_log   /var/log/gitlab/nginx/gitlab_error.log;
}
```
不用额外再配置，HTTP 会自动跳转到 HTTPS 。
开放443端口，因为防火墙未开启，此处省略。


## 三、gitLab服务管理 ##

### 3.1、服务管理 ###

```
# 启动所有 gitlab 组件：
gitlab-ctl start
# 停止所有 gitlab 组件：
gitlab-ctl stop
# 停止所有 gitlab postgresql 组件：
gitlab-ctl stop postgresql
# 停止相关数据连接服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
# 重启所有 gitlab 组件：
gitlab-ctl restart
# 重启所有 gitlab gitlab-workhorse 组件：
gitlab-ctl restart  gitlab-workhorse
# 查看服务状态
gitlab-ctl status
# 生成配置并启动服务
gitlab-ctl reconfigure
```

### 3.2、日志管理 ###

```
# 实时查看所有日志
gitlab-ctl tail

# 实时检查redis的日志
gitlab-ctl tail redis
 
# 实时检查postgresql的日志
gitlab-ctl tail postgresql
 
# 检查gitlab-workhorse的日志
gitlab-ctl tail gitlab-workhorse
 
# 检查logrotate的日志
gitlab-ctl tail logrotate
 
# 检查nginx的日志
gitlab-ctl tail nginx
 
# 检查sidekiq的日志
gitlab-ctl tail sidekiq
 
# 检查unicorn的日志
gitlab-ctl tail unicorn
```

### 3.3、备份 ###

**3.3.1、手动备份：**
gitLab默认备份的目录为  /var/opt/gitlab/backups，如果想改备份目录，可修改/etc/gitlab/gitlab.rb：

```
gitlab_rails['backup_path'] = '/data/backups'

```

修改文件后，需要
```
[root@sjq-09 ~]# gitlab-ctl reconfigure

```

备份命令:
```
[root@sjq-09 ~]# gitlab-rake gitlab:backup:create

```
该命令会在备份目录（默认：/var/opt/gitlab/backups/）下创建一个tar压缩包xxxxxxxx_gitlab_backup.tar，其中开头的xxxxxx是备份创建的时间戳，这个压缩包包括GitLab整个的完整部分。

**3.3.2、使用 crontab实现自动备份**


```
[root@sjq-09 ~]# crontab -e

# 每天2点备份gitlab数据
0 2 * * * /usr/bin/gitlab-rake gitlab:backup:create
```

可设置只保留最近7天的备份，编辑配置文件 /etc/gitlab/gitlab.rb
```
# 数值单位：秒
gitlab_rails['backup_keep_time'] = 604800
```

重新加载gitlab配置文件
```
[root@sjq-09 ~]# gitlab-ctl reconfigure
```

### 3.4、恢复 ###

假设配置文件为: /var/opt/gitlab/backups/1499444722_2018_05_28_7.4.7_gitlab_backup.tar

停止 unicorn 和 sidekiq ，保证数据库没有新的连接，不会有写数据情况。

```
# 停止相关数据连接服务
[root@sjq-09 ~]# gitlab-ctl stop unicorn
[root@sjq-09 ~]# gitlab-ctl stop sidekiq

# 指定恢复文件，会自动去备份目录找。确保备份目录中有这个文件。
# 指定文件名的格式类似：1499444722_2018_05_28_7.4.7，程序会自动在文件名后补上：“_gitlab_backup.tar”
# 一定按这样的格式指定，否则会出现 The backup file does not exist! 的错误
[root@sjq-09 ~]# gitlab-rake gitlab:backup:restore BACKUP=1499444722_2018_05_28_7.4.7

# 启动Gitlab
[root@sjq-09 ~]# gitlab-ctl start
```

### 3.5、开机自启 ###
`vim /etc/rc.d/rc.local` 添加如下内容
```
/bin/gitlab-ctl start
```

### 3.6、数据库操作 ###
连接数据库 
```
[root@sjq-09 data]# gitlab-rails dbconsole
psql (9.6.8)
Type "help" for help.

gitlabhq_production=> \l  #查看所有数据库
                                             List of databases
        Name         |    Owner    | Encoding |   Collate   |    Ctype    |        Access privileges        
---------------------+-------------+----------+-------------+-------------+---------------------------------
 gitlabhq_production | gitlab      | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres            | gitlab-psql | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0           | gitlab-psql | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/"gitlab-psql"               +
                     |             |          |             |             | "gitlab-psql"=CTc/"gitlab-psql"
 template1           | gitlab-psql | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/"gitlab-psql"               +
                     |             |          |             |             | "gitlab-psql"=CTc/"gitlab-psql"

gitlabhq_production=> \q # 退出
```

默认情况下，postgresql只允许本机访问，配置访问权限

`vim /var/opt/gitlab/postgresql/data/pg_hba.conf` 添加如下内容
```
host    all     all     0.0.0.0/0       trust
```


`vim /var/opt/gitlab/postgresql/data/postgresql.conf` 找到 listen_addresses 设置为'*'
```
listen_addresses = '*' 
```

重启 postgresql服务:

```
gitlab-ctl restart postgresql
```
