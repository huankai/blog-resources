---
title: Git 环境搭建
date: {{ date }}
author: huangkai
tags:
    - Git
---


## 安装环境： ##
操作系统: CentOS Linux release 7.3.1611 (Core)

## 安装：##
### 1、Linux 做为服务器端系统，Windows 作为客户端系统，分别安装 Git ###

### 服务器端： ###
```
[root@huangkai ~]# yum install -y git
```
安装完后，查看 Git 版本
```
[root@huangkai ~]# git --version
git version 1.8.3.1
```

### 客户端： ###
下载 Git for Windows，地址：https://git-for-windows.github.io  ，或 [点击这里](https://pan.baidu.com/s/1nxcKj5R) 下载
安装完之后，可以使用 Git Bash 作为命令行客户端。
也可以使用 tortoiseGit 客户端来管理项目： [点击这里](https://tortoisegit.org/download/) 下载
查看 Git 版本：
```
huangkai@DESKTOP-MEKIV7C MINGW64 ~/Desktop
$ git --version
git version 2.14.2.windows.2

```

### 自报家门： ###
这里要配置的是你个人的用户名称和电子邮件地址。这两条配置很重要，每次 Git 提交时都会引用这两条信息，说明是谁提交了更新，所以会随更新内容一起被永久纳入历史记录

```
huangkai@DESKTOP-MEKIV7C MINGW64 ~/Desktop
$ git config --global user.name "huangkai"
$ git config --global user.email "huankai@139.com"
```

### 2、创建 git用户，用来管理 git 服务 ###
```
[root@huangkai ~]# useradd git
[root@huangkai ~]# passwd git
Changing password for user git.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@huangkai ~]#   

```

### 3、创建 Git 仓库 ###
设置 /home/git/test.git 为 Git 裸仓库(一般以 .git结尾,裸仓库是没有工作区的，纯粹为了共享),然后把 Git 仓库的 拥有者 修改为 git用户
```
[root@huangkai ~]# cd /home/git/
[root@huangkai git]# git --bare init test.git
Initialized empty Git repository in /home/git/test.git/
[root@huangkai git]# chown -R git:git test.git
[root@huangkai git]# ll
total 0
drwxr-xr-x. 2 git git 6 Feb 12 22:10 test.git
```

### 4、客户端 clone 远程仓库　###
进入 Git Bash 命令行客户端，创建项目地址（设置在 d:/gittest）并进入此目录，然后从 Linux git 服务器上 clone 项目：
```
huangkai@DESKTOP-MEKIV7C MINGW64 /d/gittest
$ git clone git@192.168.1.90:/home/git/test.git
```

如果SSH用的不是默认的22端口，则需要使用以下的命令（假设SSH端口号是7700）：
```
huangkai@DESKTOP-MEKIV7C MINGW64 /d/gittest
$ git clone git@192.168.1.90:7700/home/git/test.git
```
当第一次连接到目标 Git 服务器时会得到一个提示：
```
The authenticity of host '192.168.1.90 (192.168.1.90)' can't be established.
ECDSA key fingerprint is SHA256:HxWUbnmfhcxBEnkYtXPyN4xwA2OCNsE8WZzOsvhMaCY.
Are you sure you want to continue connecting (yes/no)? 
```

选择 yes：
```
Warning: Permanently added '192.168.1.90' (ECDSA) to the list of known hosts.
```
此时 C:/Users/用户名/.ssh 下会多出一个文件 known_hosts，以后在这台电脑上再次连接目标 Git 服务器时不会再提示上面的语句。后面提示要输入密码，可以采用 SSH 公钥来进行验证.

### 5、客户端创建 SSH 公钥和私钥 ###
```
huangkai@DESKTOP-MEKIV7C MINGW64 /d/gittest
$ ssh-keygen -t rsa -C "huankai@139.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/huangkai/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/huangkai/.ssh/id_rsa.
Your public key has been saved in /c/Users/huangkai/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:dXmadhishVapUGRtsrpr2QLpxLv04Yg7WQ22n6H85ww huankai@139.com
The key's randomart image is:
+---[RSA 2048]----+
|         o+...   |
|        ...++.   |
|         .+=* .  |
|      o  oo+ *   |
|     o =S.. = .  |
|      B +  . .   |
|     *.=E*       |
|    oo=+B+o      |
|    ooo+==o      |
+----[SHA256]-----+

huangkai@DESKTOP-MEKIV7C MINGW64 /d/gittest

```

输入回车、此时 C:/Users/用户名/.ssh 下会多出两个文件 id_rsa 和 id_rsa.pub

id_rsa 是私钥

id_rsa.pub 是公钥

### 6、服务器端 Git 打开 RSA 认证 ###
进入 /etc/ssh 目录，编辑 sshd_config，打开以下三个配置的注释：
```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
```
保存并重启 sshd 服务：
```
[root@huangkai gittest]# systemctl restart sshd
```

由 AuthorizedKeysFile 得知公钥的存放路径是 .ssh/authorized_keys，实际上是 $Home/.ssh/authorized_keys，由于管理 Git 服务的用户是 git，所以实际存放公钥的路径是 /home/git/.ssh/authorized_keys


在 /home/git/ 下创建目录 .ssh，将.ssh 文件夹的 owner 修改为 git
```
[root@huangkai git]# mkdir .ssh
[root@huangkai git]# ls -la
total 12
drwx------. 4 git  git   89 Feb 12 22:29 .
drwxr-xr-x. 4 root root  33 Feb 12 22:06 ..
-rw-r--r--. 1 git  git   18 Aug  3  2016 .bash_logout
-rw-r--r--. 1 git  git  193 Aug  3  2016 .bash_profile
-rw-r--r--. 1 git  git  231 Aug  3  2016 .bashrc
drwxr-xr-x. 3 git  git   18 Feb 12 22:11 gittest
drwxr-xr-x. 2 root root   6 Feb 12 22:29 .ssh
[root@huangkai git]# chown -R git:git .ssh
[root@huangkai git]# ls -la
total 12
drwx------. 4 git  git   89 Feb 12 22:29 .
drwxr-xr-x. 4 root root  33 Feb 12 22:06 ..
-rw-r--r--. 1 git  git   18 Aug  3  2016 .bash_logout
-rw-r--r--. 1 git  git  193 Aug  3  2016 .bash_profile
-rw-r--r--. 1 git  git  231 Aug  3  2016 .bashrc
drwxr-xr-x. 3 git  git   18 Feb 12 22:11 gittest
drwxr-xr-x. 2 git  git    6 Feb 12 22:29 .ssh
```

将客户端公钥导入服务器端 /home/git/.ssh/authorized_keys 文件

回到 Git Bash下，导入文件,输入服务器端 git 用户的密码（就是git用户的密码）

```
huangkai@DESKTOP-MEKIV7C MINGW64 /d/gittest
$ ssh git@192.168.1.90 'cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub
git@192.168.1.90's password:

huangkai@DESKTOP-MEKIV7C MINGW64 /d/gittest
$

```

回到服务器端，查看 .ssh 下是否存在 authorized_keys 文件：
```
[root@huangkai .ssh]# ls -la
total 4
drwxr-xr-x. 2 git git  29 Feb 12 22:43 .
drwx------. 4 git git  89 Feb 12 22:39 ..
-rw-rw-r--. 1 git git 397 Feb 12 22:43 authorized_keys
[root@huangkai .ssh]# 

```
可以通过 cat 查看authorized_keys 的内容

重要：
修改 .ssh 目录的权限为 700

修改 .ssh/authorized_keys 文件的权限为 600

```
[root@huangkai git]# chmod 700 .ssh/
[root@huangkai git]# cd .ssh/
[root@huangkai .ssh]# ll
total 4
-rw-rw-r--. 1 git git 397 Feb 12 22:43 authorized_keys
[root@huangkai .ssh]# chmod 600 authorized_keys
```
客户端再次 clone :
```
huangkai@DESKTOP-MEKIV7C MINGW64 /d/gittest
$ git clone git@192.168.1.90:/home/git/gittest/.git
```

### 7、禁止 git 用户 ssh 登录服务器 ###
之前在服务器端创建的 git 用户将此用户不允许 ssh 登录服务器

编辑 /etc/passwd

找到：
```
git:x:1001:1001::/home/git:/bin/bash
```
修改为
```
git:x:1001:1001::/home/git:/bin/git-shell
```
此时 git 用户可以正常通过 ssh 使用 git，但无法通过 ssh 登录系统。



参考资料：
 https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000
https://git-scm.com/docs