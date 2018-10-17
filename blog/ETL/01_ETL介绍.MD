---
title: ETL 利器Kettle实战应用解析系列一【01_Kettle使用介绍与安装】
date: {{ date }}
author: huangkai
tags:
	- ETL
---

最近做的项目用到了ETL(Extract-Transform-Load的缩写，即数据抽取、转换、装载的过程)工具Kettle，这个工具相当好用，可以将各种类型数据作为数据流，经过处理后再生成各种类型的数据。正如其名“水壶”，将各个地方的水倒进水壶里，再用水壶倒入不同的容器。不过一来初学乍用，二来对此任务不是很感兴趣，研究的不是很深入，可能是以一种不科学的方法使用的，但观教程，常用的内容似乎也涉及到了，并且Y大说过，要善于总结，于是有了这篇，作为入门说明吧.


Kettle是一款国外开源的ETL工具，纯java编写，可以在Window、Linux、Unix上运行，绿色无需安装，数据抽取高效稳定。

Kettle 中文名称叫水壶，该项目的主程序员MATT 希望把各种数据放到一个壶里，然后以一种指定的格式流出。

Kettle这个ETL工具集，它允许你管理来自不同数据库的数据，通过提供一个图形化的用户环境来描述你想做什么，而不是你想怎么做。

Kettle中有两种脚本文件，transformation和job，transformation完成针对数据的基础转换，job则完成整个工作流的控制。
# 一、下载与安装 #

1.1、JDK 下载与安装：

Kettle需要JRE环境，需要先安装并配置好相关环境，JRE的安装与配置这里不再啰嗦。

1.2、Kettle 下载与安装 ：

下载地址： https://sourceforge.net/projects/pentaho/files/Data%20Integration/
Kettle 7 以上版本需要最低JDK版本为 8，下载7.1 zip版本后，解压到你系统指定的目录下，如果你是在windows 上运行，双击 Kettle根目录下的 **Spoon.bat** 文件即可打开 ，如果是在 Linux 上运行，执行根目录下的 **spoon.sh** 文件，会显示图形化的界面。 

# 二、案例说明： #
本系列是将第三方数据通过ETL抽取、转换、装载后保存到自己的业务系统中。
<font color='red'> **特别说明 ：本系列案例中涉及的数据、表结构只是按个人需求生成，并不作为企业真实数据使用。**</font>

## 2.1、输入数据表结构如下： ##

### 2.1.1、部门表： ###

```
CREATE TABLE `departmentv2` (
  `部门编号` varchar(255) NOT NULL,
  `部门名称` varchar(255) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 2.1.2、班级表： ###

```
CREATE TABLE `class` (
  `所属院系` varchar(255) NOT NULL,
  `班级名称` varchar(255) NOT NULL,
  `年级` varchar(50) DEFAULT NULL,
  `班级状态` char(1) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 2.1.3、学生表： ###

```
CREATE TABLE `studentv2` (
  `学号` varchar(255) NOT NULL,
  `姓名` varchar(255) NOT NULL,
  `性别` varchar(255) NOT NULL,
  `出生日期` varchar(255) DEFAULT NULL,
  `民族` varchar(255) DEFAULT NULL,
  `系部名称` varchar(255) NOT NULL,
  `专业名称` varchar(255) NOT NULL,
  `班级名称` varchar(255) NOT NULL,
  `学制` int(11) NOT NULL,
  `学籍状态` varchar(255) NOT NULL,
  `年级` int(11) NOT NULL,
  `入校日期` varchar(255) NOT NULL,
  `籍贯` varchar(255) DEFAULT NULL,
  `手机号码` varchar(255) DEFAULT NULL,
  `身份证号码` varchar(255) DEFAULT NULL,
  `政治面貌` varchar(255) DEFAULT NULL,
  `培养层次` varchar(255) NOT NULL,
  `毕业时间` varchar(255) NOT NULL,
  PRIMARY KEY (`学号`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

## 2.2、输出数据表结构如下： ##

### 2.2.1、部门输出表： ###

```
CREATE TABLE `sp_role` (
  `role_id` char(36) NOT NULL COMMENT '角色ID',
  `org_id` char(36) DEFAULT NULL COMMENT '机构ID',
  `role_name` varchar(30) NOT NULL COMMENT '角色名称',
  `role_type` char(1) DEFAULT NULL COMMENT '角色类型',
  `status` varchar(1) NOT NULL COMMENT '状态',
  `description` varchar(100) DEFAULT NULL COMMENT '角色描述',
  `last_up_time` datetime NOT NULL COMMENT '更新时间',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`role_id`),
  KEY `sp_role_org_id` (`org_id`) USING BTREE,
  CONSTRAINT `sp_role_ibfk_2` FOREIGN KEY (`org_id`) REFERENCES `sp_org` (`org_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='角色表';

```
### 2.2.2、班级输出表： ###

```

CREATE TABLE `bas_class` (
  `class_id` char(36) NOT NULL COMMENT '班级ID',
  `school_id` char(36) NOT NULL COMMENT '学校ID',
  `class_name` varchar(50) NOT NULL COMMENT '班级名称',
  `reg_years` varchar(4) NOT NULL COMMENT '级别',
  `status` char(1) NOT NULL COMMENT '班级状态',
  `last_up_time` datetime NOT NULL COMMENT '更新时间',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`class_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 2.2.3、学生输出表： ###

```
CREATE TABLE `sp_user` (
  `user_id` char(36) NOT NULL COMMENT '用户ID',
  `user_name` varchar(50) NOT NULL COMMENT '登录账号',
  `password` varchar(50) NOT NULL COMMENT '登录密码',
  `status` varchar(1) NOT NULL COMMENT '状态',
  `last_up_time` datetime NOT NULL COMMENT '更新时间',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`user_id`),
  KEY `sp_user_user_name` (`user_name`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户表';


CREATE TABLE `sp_user_info` (
  `user_info_id` char(36) NOT NULL COMMENT '用户信息详情D',
  `user_id` char(36) NOT NULL COMMENT '用户ID',
  `org_id` char(36) NOT NULL COMMENT '机构ID',
  `user_type` char(1) DEFAULT NULL COMMENT '用户类型',
  `user_source` varchar(50) DEFAULT NULL COMMENT '用户来源',
  `phone_no` varchar(20) DEFAULT NULL COMMENT '手机号',
  `email` varchar(50) DEFAULT NULL COMMENT '邮箱',
  `real_name` varchar(50) NOT NULL COMMENT '真实姓名',
  `spell_name` varchar(150) NOT NULL COMMENT '拼音名称',
  `short_name` varchar(50) DEFAULT NULL COMMENT '拼音简称',
  `avarar` varchar(150) DEFAULT NULL COMMENT '头像',
  `sex` char(1) DEFAULT NULL COMMENT '性别',
  `cert_type` varchar(2) DEFAULT NULL COMMENT '证件类型',
  `cert_no` varchar(30) DEFAULT NULL COMMENT '证件号码',
  `solar_birth` datetime DEFAULT NULL COMMENT '生日',
  `editable` int(11) NOT NULL DEFAULT '1',
  `last_up_time` datetime NOT NULL COMMENT '更新时间',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`user_info_id`),
  UNIQUE KEY `sp_user_info_user_id` (`user_id`) USING BTREE,
  KEY `sp_user_info_org_id` (`org_id`) USING BTREE,
  CONSTRAINT `sp_user_info_ibfk_1` FOREIGN KEY (`org_id`) REFERENCES `sp_org` (`org_id`),
  CONSTRAINT `sp_user_info_user_id` FOREIGN KEY (`user_id`) REFERENCES `sp_user` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户信息详情';


CREATE TABLE `sp_user_role` (
  `user_role_id` char(36) NOT NULL COMMENT '用户角色ID',
  `user_id` char(36) NOT NULL COMMENT '用户编码',
  `role_id` char(36) NOT NULL COMMENT '角色编码',
  `last_up_time` datetime NOT NULL COMMENT '更新时间',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`user_role_id`),
  KEY `sp_user_role_user_id` (`user_id`) USING BTREE,
  KEY `sp_user_role_role_id` (`role_id`) USING BTREE,
  CONSTRAINT `sp_user_id` FOREIGN KEY (`user_id`) REFERENCES `sp_user` (`user_id`),
  CONSTRAINT `sp_user_role_ibfk_1` FOREIGN KEY (`role_id`) REFERENCES `sp_role` (`role_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='用户角色关联表';

CREATE TABLE `student` (
  `student_id` char(36) NOT NULL COMMENT '学生ID',
  `school_id` char(36) NOT NULL COMMENT '学校ID',
  `user_id` varchar(36) NOT NULL COMMENT '用户ID',
  `stu_name` varchar(50) NOT NULL COMMENT '姓名',
  `stu_no` varchar(20) NOT NULL COMMENT '学号',
  `sex` char(1) NOT NULL COMMENT '性别： 1 男  2 女 9 未说明',
  `phone_no` varchar(20) DEFAULT NULL COMMENT '联系电话',
  `cert_type` char(1) DEFAULT NULL COMMENT '证件类型',
  `cert_no` varchar(20) DEFAULT NULL COMMENT '身份证号',
  `studies_type` char(1) DEFAULT NULL COMMENT '1、专科 2 、本科 3、中专',
  `current_state` char(2) DEFAULT NULL COMMENT '当前状态: 0 在读 1 毕业',
  `graduation_date` datetime DEFAULT NULL,
  `is_delete` char(1) NOT NULL,
  `last_up_time` datetime NOT NULL COMMENT '更新时间',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`student_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```


### 2.3、数据流程图如下： ###

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/00.png)


	