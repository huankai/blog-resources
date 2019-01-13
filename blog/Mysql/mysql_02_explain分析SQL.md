---
title: Mysql_02_explain
date: {{ date }}
author: huangkai
tags:
    - Mysql
---

# explain 是什么？ #

使用 explan 关键字，你可以模拟mysql优化器执行SQL查询语句，从而知道 MYSQL 是如何处理你的SQL语句的，分析你的查询语句或是表结构的性能瓶颈。

使用 explain 能干什么？
	1、查看表的读取顺序：
	2、数据读取操作的操作类型：
	3、哪些索引可以使用：
	4、哪些索引被实际使用：
	5、表之间的引用：
	6、每张表有多少行被优化器查询

使用方式：
	使用 EXPLAIN + 查询的语句，查看执行结果：

explan 字段解析：

 - id
 	select 查询的序列号，由一组数字组成。

 	id 全相同，从上到下依次执行
 	id全不同，id值越大，会越先执行，如子查询。
 	id有相同，也有不同，id值数字大的先执行，id值相同的会从上到下顺序执行，可能会出现衍生表(DERIVED)。
 - select_type
 	主要用于区别复合查询、子查询、等复杂查询，值有:
 		- SIMPLE
 			简单查询 ，查询中不包含子查询或 UNION
 		- PRIMARY
 			查询中若包含任何复杂的子部分，最外去查询则被标记为 PRIMARY.
 		- SUBQUERY
 			在select或where 列表中包含子查询。
 		- DERIVED
 			在 FROM列表中包含子查询被标记为 DERIVED(衍生) ，MYSQL会递归执行这些子查询，把结果放在临时表里。
 		- UNION 
 			若在第二个SELECT出现了 union ，则被标记为 UNION；
 			若UNION包含在FROM了句的子查询中，外层SELECT将被标记为 DERIVED
 		- UNION RESULT
 			从 UNION表获取结果的SELECT.
 - table
 	表名
 - type
 	此参数与查询效率相关，从最差到最好排序如下（一般能优化到 range 级别会有较好的性能,最好能达到 ref 级别更好）：
 		ALL > index > range > ref > eq_ref > const > system

 		- ALL
 			最差、从磁盘全表扫描，数据量达到百万级别，一定要优化。
 		- index
 			全索引扫描，从索引中读取，如查询的字段刚好是索引列，不是使用 select * from xxx。
 		- range 
 			使用 between and 或  in  或 > / < 查询。
 		- ref
 			非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回的所有匹配某个单独值的行，也有可能找到多个符合条件的行，所以他应该属于查找和扫描的混合体，如 explain select * from user where username = 'xxx' ，用户名相同的会有多个。
 		- eq_ref
 			唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或唯一索引扫描，如两表关联，其中一张表只有一条记录。
 		- const
 			表示通过索引一次就找到了，如主键或唯一索引。
 		- system
 			表中只有一行记录，这是const的特例，平时不会出现。
 - possible_keys
 	显示可能应用到这张表中的索引，一个或多个，但不一定会被查询实际使用。
 - key
 	实际使用的索引，如果为 NULL, 则没有使用索引，如果查询中使用了覆盖索引，则该索引公出现在 key 列表中，不会出现在 possible_keys 列表中。
 - ken_len
 	索引中使用的字节数，长度越短越好。
 - ref
 	显示索引的哪一列被使用了，如果可能的话，是一个常数，哪些列或常量被用于查询索引列上的值。
 - rows
 	检索的行数，值越小越好。
 - Extra
 	其它扩展属性，
 	- Using filesort
 		mysql会对数据使用一个外部的索引排序，
 	- Using temporary
 		使用了临时表保存中间结果
 	- USING index

 	- Using where
 		使用了where 过滤
 	- Using join buffer
 		使用了连接缓存
 	- impossible where
 		where子句的值总是 false,不能用来获取任何元组。
 	- select tables optimized away


 	- distinct 
 		优化 distinct 操作，在找到第一个匹配的元组后即停止找同样值的动作。

