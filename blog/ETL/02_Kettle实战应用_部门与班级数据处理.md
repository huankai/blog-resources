---
title: ETL 利器Kettle实战应用解析系列 ————【部门与班级数据处理】
date: {{ date }}
author: huangkai
tags:
	- ETL
---

#一、部门数据处理流程如下：#

## 1、创建一个转换： ##
打开ETL工具后，使用快捷键 ` Ctrl + N` 创建一个转换。

## 2、表输入： ##

2.1、点击左边 `核心对象` 面板中，打到 `表输入` 组件，直接拖拽到右边的编辑区；

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_01.png)

2.2、双击编辑区的 `表输入` 组件，弹出配置对话框，在此可以编辑步骤名称、数据库连接配置、要查询的 sql，配置好后，可以点击下方的`预览` 查看sql的执行结果。
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_02.png)

## 3、增加常量： ##
可以定义一些常量数据。点击左边 `核心对象` 面板中，打到 `常量数据` 组件，直接拖拽到右边的编辑区；按住 `Shift` 键，将 `表输入` 组件与 `增加常量` 组件相连。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_03.png)

## 4、生成随机数： ##
生成一些主键UUID，点击左边 `核心对象` 面板中，打到 `生成随机数` 组件，直接拖拽到右边的编辑区；按住 `Shift` 键，将 `增加常量` 组件与 `生成随机数` 组件相连。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_04.png)

## 5、获取系统信息： ##
获取系统的一些相关系统，如当前时间。点击左边 `核心对象` 面板中，打到 `获取系统信息` 组件，直接拖拽到右边的编辑区；按住 `Shift` 键，将 `生成随机数` 组件与 `获取系统信息` 组件相连。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_05.png)

## 6、插入/更新数据 ##
定义输出的表、执行插入与更新的条件、流字段与输出表字段的映射关系。点击左边 `核心对象` 面板中，打到 `插入/更新` 组件，直接拖拽到右边的编辑区；按住 `Shift` 键，将 `获取系统信息` 组件与 `插入/更新` 组件相连，注意：这里有两个需要 `插入/更新` 的组件，当连接两个相同的组件时，会提示是 **分发** 还是 **复制**，这里选择 **复制** 即可。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_06.png)

## 7、执行 ##
以上步骤都定义好之后，启动查看执行结果，查看日志，如果有错误需要解决。

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_07.png)


## 8、最终流程与日志截图 ##

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/01_08.png)

通过上图日志可知：此次执行总共更新 49条记录（为什么是更新？因为我已执行过一次，根据第 7 步中的查询的条件来判断，如果成立则会执行更新，不成立才执行插入）。


#二、班级数据处理流程：#

班级数据的处理步骤比上面的更简单，上面的输出有两张表，班级输出只有一张表,和上面的从`表输入` 到 `获取系统信息`的步骤一样，这里不再列出。