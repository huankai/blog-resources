---
title: Git 多账号配置SSH
date: {{ date }}
author: huangkai
tags:
    - Git
---

Windows 下Git多账号配置SSH-key的管理

所有执行命令的地方都是在管理员模式下进行，即打开cmd，也可以使用Git Bash客户端用管理员身份运行程序。

## 1、生成github.com对应的私钥与公钥 ##
在任意目录打开 cmd 或 Git Bash ，执行命令 `ssh-keygen -t rsa -C youEmail ` 创建github对应的sshKey，命令为`id_rsa_github`

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/15.png)

会在当前目录下生成 id_rsa_github 和 id_rsa_github.pub 两个文件，将这两个文件Copy 到 `C:\Users\用户名\.ssh` 目录中。

以同样的方式生成另一个git的私钥与公钥，(邮箱地址可以相同，也可以不同，在 2 处输入生成的文本不同)，也将生成的两个文件Copy 到`C:\Users\用户名\.ssh` 目录中。

## 2、把github对应的公钥和另一个生成的git对应的公钥上传到服务器。 ##
GitHub SSH 生成位置在 `User -> Settings -> SSH and GPG keys -> New SSH Key ` 将对应的github 的 id_rsa_github.pub文件内容复制到 Key文本框中。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/16.png)

另一个git的 SSH我使用的个人服务器，所以就没有截图，如果使用的是 `gitee.com` 这个网站，也和github类似。

## 3、在.ssh目录创建 config 文件并配置(最核心的地方) ##

每个账号单独配置一个Host，每个Host要取一个别名，每个Host主要配置HostName和IdentityFile两个属性即可

Host的名字可以取为自己喜欢的名字，不过这个会影响git相关命令，例如：
Host mygit 这样定义的话，命令如下，即git@后面紧跟的名字改为 mygit
git clone git@mygit:huankai/hk-core.git

 

HostName 　　　　　　　   这个是真实的域名地址
IdentityFile 　　　　　　　  这里是id_rsa的地址
PreferredAuthentications   配置登录时用什么权限认证--可设为publickey,password keyboard-interactive等
User 　　　　　　　　　　　配置使用用户名


```
# 配置github.com
Host github.com                 
    HostName github.com
    IdentityFile C:\\Users\\huangkai\\.ssh\\id_rsa_github
    PreferredAuthentications publickey
    User huankai

# 配置个人服务器
Host 192.168.1.100 
    HostName 192.168.1.100
    IdentityFile C:\\Users\\huangkai\\.ssh\\id_rsa_my
    PreferredAuthentications publickey
    User huangkai
```

## 4、打开Git Bash客户端执行测试命令测试是否配置成功（会自动在.ssh目录生成known_hosts文件把私钥配置进去） ##

```
sjq-278@DESKTOP-V9JFELK MINGW64 /e
$ ssh -T git@github.com
Hi huankai! You've successfully authenticated, but GitHub does not provide shell access.

sjq-278@DESKTOP-V9JFELK MINGW64 /e

```
如上，测试成功后，就可以使用不同的git ssh操作了，互相不会影响。
如果你使用的了TortoiseGit 管理工具，在自动 Load Putty Key 时，要选择对应的 ppk文件，ppk文件的生成，请点击 [这里](https://huankai.github.io/2018/03/26/Git_02_%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/) 查看。