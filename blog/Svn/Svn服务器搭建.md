---
title: Linux Svn 服务器搭建
date: {{ date }}
author: huangkai
tags:
    - Linux
    - Svn
---
# 1. 检查svn版本是否有安装 #
```
rpm -qa subversion
```
执行上面的命令，没有任何显示，表示没有安装，如果有，执行 **yum remove subversion** 删除。
# 2. 执行安装 #
```
yum -y install subversion
```
等待安装完成即可

# 3. 配置和启动svn服务器 #

- 创建目录 ：
```
mkdir -p /data/svn/data   # svn数据保存目录 -p 参数表示如果子目录不存在，也会级联创建目录
mkdir -p /data/svn/passwd   #svn公共用户密码与权限目录
```
- 启动 svn服务：
```
svnserve -d -r /data/svn/data/
-d	守护进程启动 
-r	指定svn目录
```
- 查看服务进程：
```
ps -ef|grep svn
root      1748     1  0 21:44 ?        00:00:00 svnserve -d -r /data/svn/data/
root      1771  1401  0 21:45 pts/0    00:00:00 grep --color=auto svn
```
如上，表示svn服务已启动

- 查看端口号：
&nbsp;&nbsp;svn默认使用的端口号为 3690
```
lsof -i:3690
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
svnserve 1748 root    3u  IPv4  24804      0t0  TCP *:svn (LISTEN)
```
或者使用如下方式查看
```
netstat -lntup|grep 3690
tcp        0      0 0.0.0.0:3690            0.0.0.0:*               LISTEN      1748/svnserve 
```

# 4. 创建svn仓库 #
```
[root@huangkai200 data]# svnadmin create hk   #创建仓库， hk 是仓库名称
[root@huangkai200 data]# ls
hk
[root@huangkai200 data]# cd hk/
[root@huangkai200 hk]# ll  # 查看仓库目录结构
total 16
drwxr-xr-x 2 root root   51 Mar 12 21:58 conf   # svn仓库配置文件 
drwxr-sr-x 6 root root 4096 Mar 12 21:58 db   #数据存放文件
-r--r--r-- 1 root root    2 Mar 12 21:58 format 
drwxr-xr-x 2 root root 4096 Mar 12 21:58 hooks
drwxr-xr-x 2 root root   39 Mar 12 21:58 locks
-rw-r--r-- 1 root root  229 Mar 12 21:58 README.txt
[root@huangkai200 hk]# cd conf/ 
[root@huangkai200 conf]# ll 
total 12
-rw-r--r-- 1 root root 1080 Mar 12 21:58 authz    #权限管理文件 
-rw-r--r-- 1 root root  309 Mar 12 21:58 passwd    #用户与密码文件 
-rw-r--r-- 1 root root 3090 Mar 12 21:58 svnserve.conf  #主配置文件，包含上面authz 与 passwd文件
```

# 5、修改 svnserver.conf 主配置文件 #
```
[root@huangkai200 conf]# vi svnserve.conf
### This file controls the configuration of the svnserve daemon, if you
### use it to allow access to this repository.  (If you only allow
### access through http: and/or file: URLs, then this file is
### irrelevant.)

### Visit http://subversion.apache.org/ for more information.
 
[general]
### The anon-access and auth-access options control access to the
### repository for unauthenticated (a.k.a. anonymous) users and
### authenticated users, respectively.
### Valid values are "write", "read", and "none".
### Setting the value to "none" prohibits both reading and writing;
### "read" allows read-only access, and "write" allows complete
### read/write access to the repository.
### The sample settings below are the defaults and specify that anonymous
### users have read-only access to the repository, while authenticated
### users have read and write access to the repository.
# anon-access = read #将此项目默认配置改为 none，svn默认对未认证的用户有只读权限，我们要将此用户的权限设置为 none，然后将 20行 的 auth-access 的注解删除
anon-access = none    
auth-access = write
### The password-db option controls the location of the password
### database file.  Unless you specify a path starting with a /,
### the file's location is relative to the directory containing
### this configuration file.
### If SASL is enabled (see below), this file will NOT be used.
### Uncomment the line below to use the default password file.
# svn默认每个仓库会指定密码文件 ，为了多个仓库统一密码管理 ，将此选项设置为公共的密码文件
password-db = /data/svn/passwd/passwd
### The authz-db option controls the location of the authorization
### rules for path-based access control.  Unless you specify a path
### starting with a /, the file's location is relative to the the
### directory containing this file.  If you don't specify an
### authz-db, no path-based access control is done.
### Uncomment the line below to use the default authorization file.
authz-db = /data/svn/passwd/authz  #和密码一样，将此选项设置为公共的权限文件
### This option specifies the authentication realm of the repository.
### If two repositories have the same authentication realm, they should
### have the same password database, and vice versa.  The default realm
### is repository's uuid.
# realm = My First Repository
```
# 6、拷贝authz  和 passwd 文件 到 公共用户与密码的目录 #
```
[root@huangkai200 hk]# cp authz passwd /data/svn/passwd
[root@huangkai200 passwd]# ll # 可以看到，如下两个文件的权限为 644（文件所有者可读可写，所在组可读，其它组可读，这样，其它用户就可以看到这两个文件 ，文件的安全性就降低了）
total 8
-rw-r--r-- 1 root root 1080 Mar 12 22:27 authz
-rw-r--r-- 1 root root  329 Mar 12 22:25 passwd
[root@huangkai200 passwd]# 

 修改文件权限：

[root@huangkai200 passwd]# chmod 600 * #(给文件所有者可读写权限，其它用户没有权限)
[root@huangkai200 passwd]# ll
total 8
-rw------- 1 root root 1080 Mar 12 22:27 authz
-rw------- 1 root root  329 Mar 12 22:25 passwd
```

# 7、SVN添加账户 #
在 passwd文件中[users]下按例子添加账户
```
[root@huangkai200 passwd]# vi passwd

### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.
[users]
# harry = harryssecret
# sally = sallyssecret
huangkai = huangkai  #添加账号，等号前为用户名，等号后为 密码
```

# 8、修改authz配置文件 #
```
[root@huangkai200 passwd]# vi authz

### This file is an example authorization file for svnserve.
### Its format is identical to that of mod_authz_svn authorization
### files.
### As shown below each section defines authorizations for the path and
### (optional) repository specified by the section name.
### The authorizations follow. An authorization line can refer to:
###  - a single user,
###  - a group of users defined in a special [groups] section,
###  - an alias defined in a special [aliases] section,
###  - all authenticated users, using the '$authenticated' token,
###  - only anonymous users, using the '$anonymous' token,
###  - anyone, using the '*' wildcard.
###
### A match can be inverted by prefixing the rule with '~'. Rules can
### grant read ('r') access, read-write ('rw') access, or no access
### ('').
[aliases]
# joe = /C=XZ/ST=Dessert/L=Snake City/O=Snake Oil, Ltd./OU=Research Institute/CN=Joe Average
[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
# [/foo/bar]
# harry = rw
# &joe = r
# * =
# [repository:/baz/fuz]
# @harry_and_sally = rw
# * = r
[hk:/]
huangkai = rw
#权限配置规则
#[版本库:/项目/目录]
#用户名 = rw(可写可读) | r (只读)
#也可以指定组，一个组对应一个或多个用户多个用户用 逗号隔开
# gruop_1 = huangkai,...
#为组配置权限规则,如 ： @ 组名
# @gruop_1 = rw
```

修改 authz 与 passwd文件 不需要重启svn服务器，但修改 svnserver.conf 必须重启使配置生效，
authz 与 passwd 配置文件不能写错，否则服务可能不能正常启动。
- 启动服务:

```
[root@huangkai200 passwd]# 

lsof -i:3690    #先查看服务是否启动
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
svnserve 1748 root    3u  IPv4  24804      0t0  TCP *:svn (LISTEN)

root@huangkai200 passwd]# kill -9 1748		#如果有启动，先关闭
[root@huangkai200 passwd]# ps -ef|grep svn
root      3252  1401  0 22:58 pts/0    00:00:00 grep --color=auto svn
[root@huangkai200 passwd]# svnserve -d -r /data/svn/data/	#启动服务 
```

# 9、svn 开机启动 #
首先：编写一个启动脚本svn_startup.sh，我放在/data/svn/svn_startup.sh
```
#!/bin/bash
/usr/bin/svnserve -d -r /data/svn/data/
```
这里的svnserve路径保险起见，最好写绝对路径，因为启动的时候，环境变量也许没加载。
绝对路径怎么查？使用 **which svnserve** 查看：
```
[root@huangkai200 hk]# which svnserve 
/usr/bin/svnserve
```
然后修改该脚本的执行权限：
```
[root@huangkai200 svn]# chmod 700 svn_startup.sh	
```
最后：加入自动运行
```
[root@huangkai200 svn]#vi /etc/rc.d/rc.local
在末尾添加脚本的路径，如：/data/svn/svn_startup.sh
```
看看 /etc/rc.d/rc.local的权限，如果没有可执行权限，则需要设置 ， chmod 711 /etc/rc.d/rc.local

# 10、svn 客户端连接 ： #
确保防火墙关闭或开放 3690 端口，现在，你可以重启一下试试了
```
svn://ip地址:3690/hk
```



 


