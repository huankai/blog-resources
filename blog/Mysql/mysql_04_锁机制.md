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
事务是由一组SQL语句组成的逻辑处理单元，事务具有以下4个属性，通常简称为事务的ACID属性：
		- 原子性(Atomicity)
事务是一个原子操作单元，其对数据的修改，要么全部执行，要么全部不执行。
		- 一致性(Consistent)
在事务开始和完成时，数据都必须保持一致性状态，这意味着所有相关的数据规则都必须应用于事务的修改，以保证数据的完整性；事务结束时，所有内部的数据结构都必须是正确的。
		- 隔离性(Isolation)
数据库系统一定的隔离机制，保证事务在不受外部并发影响的“独立”环境运行，这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。
		- 持久性(Durable)
事务完成之后，它对于数据的修改是永久性的，即使出现系统故障也能够保持。


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



# 三、行锁分析 #

```
CREATE TABLE `book` (
  `id` varchar(36) NOT NULL,
  `category_id` varchar(36) DEFAULT NULL,
  `name` varchar(50) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `category_id` (`category_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

## 3.1、行锁演示 ##

如下：行锁演示，使用 InnoDB 引擎，两个会话操作同一条记录，如果不是操作的同一条记录，则不会出现阻塞现象：
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_35.png)


## 3.2、行锁变表锁 ##
如下：mysql做强制类型转换，会导致行锁变成表锁。
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_36.png)


## 3.3、间隙锁 ##
什么是间隙锁？
当我们用范围条件而不是相等条件检索数据，请请求共享或排它锁时， InnDB 会给符合条件的已有数据记录的索引项加锁，对于键值在条件范围内但不存在的记录，叫做 “间隙(GAP)”。InnoDB 也会对这个"间隙"回销，这种锁机制就是所谓的间隙锁(Next-Key锁)。

间隙锁的危害：
当锁定一个范围键值后，即使这引起不存在的键值也会无辜的锁定，造成在锁定的时候无法插入锁定键值范围内的任何数据，在某些场景下这可能会对性能造成很大的危害。

如下：在更新一个范围内的记录时，mysql会锁定此范围内的所有记录，即使此记录不存在，所以，一般情况下，在执行更新或删除操作时，尽量使用等于。
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_37.png)

## 3.4、锁定某一行 ##

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/Mysql/mysql_38.png)

## 3.5、案例结论 ##

InnoDB 存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会高，但在并发处理能力方面要远远估于 MyISAM 的表级锁定，当系统并发量较高时，InnDB 的整体性能比 MyISAM 的性能要高。但是，InnoDB 的 行级锁定同样也有其脆弱的一而，当我们使用不当的时候，可能会让InnoDB的整体性能表现不及 MyISAM ，甚至更差。

## 3.5、查看 InnoDB 参数 ##

```
mysql> SHOW STATUS LIKE "innodb_row_lock%";
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 0     |
| Innodb_row_lock_time_avg      | 0     |
| Innodb_row_lock_time_max      | 0     |
| Innodb_row_lock_waits         | 0     |
+-------------------------------+-------+
5 rows in set (0.03 sec)

mysql> 
```
参数解释如下，红色表示重要参数，如果值比较大时，需要考虑优化：
- Innodb_row_lock_current_waits 当前正在等待锁定的数量
- <b style='color:red;'>Innodb_row_lock_time</b>	从系统启动到现在锁定总时间长度
- <b style='color:red;'>Innodb_row_lock_time_avg</b>	每次等待所花平均时间
- Innodb_row_lock_time_max	从系统启动到现在等待最长的一次所花时间
- <b style='color:red;'>Innodb_row_lock_waits</b>		系统启动后现在总共等待的次数

## 3.6、优化建议 ##
- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。
- 合理设计索引，尽量缩小索引的范围
- 尽可能较少检索条件，避免间隙锁
- 尽量控制事务大小，减少锁定资源量的时间长度
- 尽可能低级别事务隔离

## 3.7、页锁 ##
开销和加锁时间介于表锁与行锁之间，会出现死锁，锁定粒度介于表锁与行锁之间，并发一般。