---
title: Git 基本使用（TortoiseGit）
date: {{ date }}
author: huangkai
tags:
    - Git
---

# 一、	安装与基本配置 #
## 1.1、下载与安装： ##.
Git for windows 下载地址：
https://github.com/git-for-windows/git/tags
选择需要的版本下载，下载完成后，双击安装，使用默认的安装即可，安装完成后，右键菜单就会出现 **Git GUI**  和 **Git Bash** 两个快捷菜单，Git GUI 我们一般不用，它没命令方式强大，一般情况我们会使用Git Bash 操作。

## 1.2、自报家门： ##
Git 安装完成后，需要配置你个人的用户名和电子邮箱，这两个配置很重要，每次Git 提交时都会引用这两个配置信息，指明是谁提交了，也会随更新内容一起被永久纳入历史记录。

在你电脑的任意位置，<font color='red'>**右键 --> Git Bash**</font> ，会弹出 Git 命令框，在命令框中设置用户名与邮箱如下：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/01.png)

此配置会在你的电脑 <font color='red'>C:\Users\huangkai</font>目录下生成 <font color='red'>.gitconfig</font> 文件，没错，这个文件的内容就是存储以上两条命令的结果。注意，这个目录是当前登陆用户名的目录 ，我登陆的账号就是 huangkai ，如果你是用 administrator 登陆的，那这个文件应该在 C:\Users\administrator 这个目录下。

还可以使得<font color='red'>**git config –list**</font> 查看Git 的参数，如下,就有上面配置的用户名与邮箱：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/02.png)

# 二、基本操作(使用 TortoiseGit) #

## 2.1、安装TortoiseGIt： ##
tortoiseGit 客户端下载地址：
https://tortoisegit.org/download/

打开此页面后，会显示最新版本下载，这里可根据你本机安装的git版本进行下载，如果你安装的是Git 版本是2.15 ，可点击 PREVIEW RELEASE 查看之前的版本进行下载，下载完成后，直接下一步，安装即可。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/03.png)

语言包支持：
在此下载页面下方会有各语言包支持，如果你有英文恐惧症，还是资深患者，可以根据你的系统位数下载对应的中文包。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/04.png)

下载语言包后，安装完成，然后右键 TortoiseGit  Settings 中设置对应的语言包，如下图所示

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/05.png)


## 2.2、克隆远程到本地库 ##
需要输入的参数：
URL : 克隆的地址
Directory: 保存在本地的目录
Branch: 分支名称，默认为 master
Load Putty Key:自动加载 ppk文件，如果不选择，你需要启动 pageant这个程序，并将 ppk文件导入到pageant中，建议选中此项，并选择ppk文件所在目录。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/06.png)

如何生成ppk文件？
- 1、先生成 RSA 公钥与私钥：
右键打开git bash，在控制台中输入以下命令：
```
$ ssh-keygen -t rsa -C "youremail@example.com"
```
然后按几次回车，就会在你电脑的C:\Users\huangkai\.ssh 目录下生成 id_rsa (私钥)与 id_rsa.pub(公钥)这两个文件。

- 2、在开始菜单中搜索，PuttyKen 双击打形式这个程序
<font color='red'>Conversions --> ImportKey --> 选择生成RSA 私钥的文件--> Save private key</font> ，保存这个生成的文件到指定的目录即可。

## 2.3、提交 ##
当本地文件有修改，需提交到远程服务器时，进入项目根目录，右键点击
Git Commit ->’branchName’ 这个菜单，branchName 是你当前所在分支的名称，输入提交的消息，选择需要提交的文件，然后点击下面的 Commit & Push 提交并推送到远程服务器。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/07.png)

## 2.4、更新 ##
<font color='red'>**右键 TortoiseGit --> Pull**</font>

或者 <font color='red'>**右键 TortoiseGit --> Fetch**</font>

两者区别：
Pull: 拉下更新后会自动合并本地分支,其实是 Fetch 与 merge两个命令的合并。

Fetch: 先把更新拉下来，此时工作区不会自动合并,在用merge或rebase进行合并。

## 2.5、删除 ##
如果你要删除一个没有加入到Git管理的文件或文件夹，那你可以直接删除即可，
如果你要删除一个已加入到Git管理的文件或文件夹，可以先直接删除这个文件，然后提交，
比如，删除readme.md这个文件，如下，可以看出这个文件的状态为 missing

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/08.png)

你也可以选择需要删除的文件，右键<font color='red'>** TortoiseGit --> Delete --> Remove**</font> ，此时，选中的文件就会从本地工作区删除，再执行提交操作如下：可以看出该文件的状态为Deleted.

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/09.png)

## 2.6、版本回退 ##

在项目根目录下右键<font color='red'>**TortoiseGit --> Show log**</font> ，弹出对话框如下，显示的是所有提交的历史记录

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/10.png)


每点击每个提交的记录，在下方会有本次提交的修改，如果你要回退到这个版本，可以选择这次提交的记录后 右键  Reset “master” to this 弹出如下：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/11.png)

reset Type:
- soft: 软重置，不会改变工作区和索引
- mixed: 混合，保持工作区不变，重置索引文件
- hard：重置工作区和索引，会丢弃在本地的变更

选择<font color='red'>**Hard --> Ok**</font> ，此时，你的工作区应该就回退到此版本了。如果你再使用 
<font color='red'>**TortoiseGit --> Show log**</font> 查看日志记录，只会显示到回退版本的提交记录。
注意，此时你要提交修改到远程服务器，会提交失败，因为你的工作区不是最新的版本
如果你又想回到最后提交的版本，在项目根目录下 右键 <font color='red'>**TortoiseGit --> Show reflog**</font>，会显示各种版本号，选择你需要指定退回的版本号id ，<font color='red'>**右键 --> Reset “master” to this**</font> ，此时，你的工作区又是当前最新的版本。


## 2.7、撤消修改 ##
选择需要撤消修改的文件，右键 <font color='red'>**TortoiseGit --> Revert**</font>
注意，此操作会丢失本地的修改，无法找回，请慎用！！！

## 2.8、解决冲突 ##

测试例子：
Readme.md文件内容如下：
```
SDFSDFADFADFS
```

UserA 修改readme.md文件，并提交
```
SDFSDFADFADFS
abceda
```

UserB 也修改了readme.md文件
```
SDFSDFADFADFS
hello
```

此时，用户B想要提交文件，先更新项目，更新项目时，会报错，如下：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/12.png)

解决办法：
- 1、先把自己的代码保存一个版本隐藏起来
右键 <font color='red'>**TortoiseGit --> Stash Save**</font>  输入消息后保存。保存成功后，此时你项目的所有文件都应该是绿色的勾
- 2、重新更新项目
<font color='red'>**右键TortoiseGit --> pull**</font>
此时更新下的项目就是UserA 服务器上最新的项目
- 3、把隐藏的代码取回来
<font color='red'>**右键 TortoiseGit --> Stash Pop**</font>
此时会弹出框告诉你代码有冲突，有冲突的文件也变成了黄色感叹号，并询问是否需要解决它 
- 4、编辑冲突
上一步点击是后，会弹出如下冲突列表对象框，你需要双击每个文件依次解决冲突

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/13.png)

双击readme.md文件，弹出冲突编辑对话框如下

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Git/14.png)

如上，红色的行表示为有冲突的地方，你可以光标指定批有冲突的地方，右键会弹出菜单：
Use this text block: 使用此文本块
Use this whole file: 使用整个文件
Use text block from right before left 优先使用左边文本块
Use text block from left before right 优先右边文本块

可根据你的选择进行编辑冲突，最后要保证下面合并后的内容不能有红色的行，然后点击菜单栏中的的 **mark as resolved**(标记为解决)。关闭此窗口，冲突解决完成。然后，你就可以继续提交你本地的修改了。









