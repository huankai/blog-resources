---
title: Mysql_04_锁机制
date: {{ date }}
author: huangkai
tags:
    - Mysql
---

锁是计算机协调多个进程或线程并发访问某一资源的机制。
在数据库中，除了传统的计算资源(如 CPU、RAM、I/O 等)的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤为重要，也更加复杂。

# 一、Mysql 锁的分类 #

- 从对数据库操作的类型来分
	-	读锁(共享锁)
针对同一份数据，多个读操作可以同时进行而不会互相影响。
	-	写锁(排它锁)
当前写操作没有完成前，它会阻断其它写锁和读锁。

- 从对数据操作的粒度来分
	- 表锁
偏向于 MYISAM 存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
	- 行锁
偏向于 InnoDB 存储引擎，开销大，加锁慢，会出现死锁，锁定的粒度小，发生锁冲突的概率最低，并发度高。InnoDB 与 MyISAM 最大不同有两点： 一是事务支持，二是使用了行级锁。

# 二、表锁分析 #

```
CREATE TABLE `mylock` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

CREATE TABLE `book` (
  `id` varchar(36) NOT NULL,
  `category_id` varchar(36) DEFAULT NULL,
  `name` varchar(50) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `category_id` (`category_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

insert into mylock(name) values("a");
insert into mylock(name) values("b");
insert into mylock(name) values("a");
insert into mylock(name) values("c");
insert into mylock(name) values("d");
insert into mylock(name) values("e");

```
## 2.1、查看锁定的表 ##
```
mysql> show open tables where In_use > 0;
Empty set

mysql> 
```
## 2.2、锁定表与解锁表 ##
```
################ 锁定  mylock 读，book 表写
mysql> lock table mylock read,book write;
Query OK, 0 rows affected (0.01 sec)

################ 查看指定的表
mysql> show open tables where In_use > 0;
+----------+--------+--------+-------------+
| Database | Table  | In_use | Name_locked |
+----------+--------+--------+-------------+
| db0114   | book   |      1 |           0 |
| db0114   | mylock |      1 |           0 |
+----------+--------+--------+-------------+
2 rows in set (0.06 sec)

################ 解锁所有表
mysql> unlock tables; 
Query OK, 0 rows affected (0.01 sec)

################ 再次查看锁定的表
mysql> show open tables where In_use > 0;
Empty set

mysql> 
```

## 2.3、测试 ##

- 锁定读操作
如下图：session1 锁定 mylock 读操作时  session1 可以查询锁定的表，但不能更新锁定的表，也不能查询其它的表，也不能更新其它的表，sessioin2 可以查询session1锁定的或未锁定的表，可以更新 session1未锁定的表，但如果更新 session1中锁定的表时，会出现阻塞现象，需要先将session1 锁定的表解锁。
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_33.png)

- 锁定写操作
如下，session1 锁定 mylock 表写操作时， session1 可以查询或修改锁定的表，但不能查询或修改其它的表， session2 可以查询或修改未锁定的表，但查询或修改锁定的表时会出现阻塞现象，需要将 session1 先解锁。
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_34.png)

MYISAM 在执行查询语句(SELECT)时，会自动给涉及到所有表加读锁，在执行增删改操作前，会自动给涉及的表加写锁，MySQL的表级锁有两种模式：
1、表共享读锁
2、表独占写锁

如何分析表锁定？
可以通过检查 Table_locks_immediate 和 Table_locks_waited 状态变量来分析系统上的表锁定。

```
mysql> show status like 'table_locks%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 29    |
| Table_locks_waited    | 0     |
+-----------------------+-------+
2 rows in set (0.03 sec)

mysql> 
```
Table_locks_immediate: 产生表级锁定的次数，表示可以立即获取锁表的查询次数，每立即获取锁值加 1
Table_locks_waited:	出现表级锁定争用而发生等待的次数，此值高说明存在较严重的表级锁争用情况。

此外，MyISAM 的读写锁调度是写优先，这也是MyISAM不适合做写为主的引擎，因为写锁后，其它线程不能做任何事情，大量的更新会使查询很难得到锁，从而造成永远阻塞。


