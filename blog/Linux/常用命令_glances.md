---
title: linux 监控工具 glances
date: {{ date }}
author: huangkai
tags:
    - Linux
---

glances 为 Unix 和 Linux 性能专家提供监视和分析性能数据的功能相比 htop更为强大，其中包括：

- CPU 使用率
- 内存使用情况
- 内核统计信息和运行队列信息
- 磁盘 I/O 速度、传输和读/写比率
- 文件系统中的可用空间
- 磁盘适配器
- 网络 I/O 速度、传输和读/写比率
- 页面空间和页面速度
- 消耗资源最多的进程
- 计算机信息和系统资源

# 安装 #
安装环境 ：Centos 7

安装 glances :**sudo yum install glances -y**

安装完成后，在命令行执行 ``glances`` 如下：
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Linux/glances_01.png)


glances 标准模式(直接在cmd 输入 glances)运行界面快捷键使用说明:

|字符|描述|
|:--:|:--:|
|``n``|显示或隐藏网络 IO信息|
|``d``|显示或隐藏磁盘 IO信息|
|``f``|显示或隐藏磁盘 大小信息|
|``c``|按照cpu使用情况排序显示进程名列表|
|``m``|按照 内存使用情况排序显示进程名列表|
|``u``|按照 用户排序显示进程名列表|
|``Enter``|输入进程名关键字，再按 Enter 只显示指定的进程名信息|
|``Esc``|退出运行界面|

# 通过 glances 输出颜色了解系统性能 #

- 绿色表示性能良好，无需做任何额外工作；（此时 CPU 使用率、磁盘空间使用率和内存使用率低于 50%，系统负载低于 0.7）。

- 蓝色表示系统性能有一些小问题，用户应当开始关注系统性能；（此时 CPU 使用率、磁盘空间使用率和内存使用率在 50%-70% 之间，系统负载在 0.7-1 之间）。

- 品红表示性能报警，应当采取措施比如备份数据；（此时 CPU 使用率、磁盘空间使用率和内存使用率在 70%-90% 之间，，系统负载在 1-5 之间）。

- 红色表示性能问题严重，可能宕机；（此时 CPU 使用率、磁盘空间使用率和内存使用率在大于 90%，系统负载大于 5）。


# 其它 #
glances GitHub： https://github.com/nicolargo/glances
glances 官网： https://glances.readthedocs.io/en/stable/