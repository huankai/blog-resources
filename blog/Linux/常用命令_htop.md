---
title: htop
date: {{ date }}
author: huangkai
tags:
    - Linux
---

htop 的界面字符解释和 top 差不多，可以参照: [https://huankai.github.io/2018/04/23/常用命令_top](https://huankai.github.io/2018/04/23/常用命令_top)

# 安装 #
安装环境 ：Centos 7

安装扩展源:**sudo yum install epel-release -y**
安装top:**sudo yum install htop -y**

安装完成后，在命令行执行 ``htop`` 如下：
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Linux/htop_01.png)


|字符|描述|
|:--:|:--:|
|``F1 Help``|帮助|
|``F2 Setup``|设置（可以设置显示的参数，方式等|
|``F3 Search``|搜索(光标跳到含有输入字符的行)|
|``F4 Filter``|筛选（只保留完全匹配输入字符的行）|
|``F5 Tree``|查看进程数(和pstree差不多)|
|``F6 Sorted``|排序|
|``F7 Nice -``|增加优先级|
|``F8 Nice +``|减少优先级|
|``F9 Kill``|杀死进程，默认信号15|
|``F10 Quit``|退出|