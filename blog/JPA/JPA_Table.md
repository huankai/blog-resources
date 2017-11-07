---
title: JPA @Table 注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

@Table 是类级别的注解，用于声明实体映射到数据库中的具体的表。

|参数|类型|描述|
|:---:|:---:|:---:|
|name|String|表的名称，默认为实体名称（参考 @Entity 注解的 name 参数说明），因此如果实体名称与映射的表名称一致时，@Table 注解常常可以省略|
|catalog|String|默认为数据库系统缺省的 catalog|
|schema|String|默认为用户缺省的 schema|
|indexes|Index[]|表的索引，通过使用 @Index 注解来声明，仅在允许自动更新数据库表结构的场景中起到作用，默认没有其他额外的索引|
|uniqueConstraints|UniqueConstraint[]	|表的唯一约束（除了由 @Column 和 @JoinColumn 注解指定的约束以及主键的约束之外的约束），通过使用 @UniqueConstraint 注解来声明，仅在允许自动更新数据库表结构的场景中起到作用，默认没有其他额外的约束条件|

## 1、catalog 和 schema 的区别 ##

catalog 和 schema 主要用来解决数据库系统命名冲突的问题。一个数据库系统可以包含多个 catalog，每个 catalog 可以包含多个 schema，而每个 schema 又可以包含多个数据库对象（表、视图等）。不同的数据库系统对 catalog 和 schema 的支持方式有所不同，常见的数据库系统：

|数据库系统|catalog|schema|
|:---:|:---:|:---:|
|MySQL|	不支持|	数据库名|
|Oracle	|不支持|	用户 ID|
|SQLServer|	数据库名|	对象属主名|
|DB2	|指定数据库对象时，Catalog 可以省略	Catalog |属主名|
|Sybase|	数据库名	|数据库属主名|

## 2、 唯一约束和索引的区别 ##
唯一约束是用来确保数据的正确性，它不允许表中存在重复的数据，若新插入的数据在表中已经存在，则更新操作失败。在数据库系统中，创建一个唯一约束的同时，也会为该约束所指定的所有列创建一个唯一索引，即约束包含索引。

索引是用来优化数据库表数据的检索性能的。通常，出现在查询 SQL 的 WHERE 子句和 JOIN 子句中的列可以考虑为其建立索引。

## 3、 @UniqueConstraint ##

用于声明表的唯一约束，这些仅在允许自动更新数据库表结构的场景中起到作用。

|参数|类型|描述|
|:---:|:---:|:---:|
|name|String|约束名称，如果不指定，默认使用数据库提供商所生成的值|
|columnNames|String[]|约束的列名称|

## 4、 @Index ##

用于声明表的索引，这些仅在允许自动更新数据库表结构的场景中起到作用。另外，不需要为表的主键指定索引，因为主键索引会自动被创建。

|参数|类型|描述|
|:---:|:---:|:---:|
|name|String|索引名称，如果不指定，默认使用数据库提供商所生成的值|
|columnList|String|要包含在索引中的列名称|
|unique|boolean|索引是否唯一，默认为 false|

## 5、@Table ##

### 5.1 唯一约束 ###

name 列、mail 列的值必须是唯一的，不允许出现重复的值：
```
@Entity(name = "user")
@Table(uniqueConstraints = {
        @UniqueConstraint(name = "unique_name", columnNames = "name"),
        @UniqueConstraint(name = "unique_mail", columnNames = "mail")
})
public class User implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_name` (`name`),
  UNIQUE KEY `unique_mail` (`mail`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 5.2。 联合唯一约束 ###

多列联合唯一约束，name 列和 mail 列不能同时出现相同的值：

```
@Entity(name = "user")
@Table(uniqueConstraints = @UniqueConstraint(name = "unique_name_mail", columnNames = {"name", "mail"}))
public class User implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_name_mail` (`name`,`mail`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 5.3、 单列索引 ###

为 name 列和 mail 列分别建立索引

```
@Entity(name = "user")
@Table(indexes = {
        @Index(name = "index_name", columnList = "name"),
        @Index(name = "index_mail", columnList = "mail")
})
public class User implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_name` (`name`),
  KEY `index_mail` (`mail`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 5.4、 多列索引 ###

为 name 列和 mail 列建立多列索引：

```
@Entity(name = "user")
@Table(indexes = @Index(name = "index_name_mail", columnList = "name,mail"))
public class User implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_name_mail` (`name`,`mail`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 5.5、 单列索引和多列索引的区别 ###

当 SQL 查询条件中包含 name 和 mail 时：

```
SELECT * FROM user WHERE name = 'admin' AND mail = 'admin@163.com'
```
如果为 name 和 mail 列分别建立索引，当执行查询时，MySQL 只能使用一个索引。如果发现有多个单列索引可用，MySQL 会试图选择一个限制最严格的索引来检索，而其他索引则利用不上。
使用分析器分析查询 SQL：

```
EXPLAIN SELECT * FROM user WHERE name = 'admin' AND mail = 'admin@163.com'
```
结果如下：

```
+--------+------+-----------------------+------------+---------+-------+------+-------------+
| id | select_type | table  | type | possible_keys         | key        | key_len | ref   | rows | Extra       |
+----+-------------+--------+------+-----------------------+------------+---------+-------+------+-------------+
|  1 | SIMPLE      | user | ref  | index_name,index_mail | index_mail | 768     | const |    1 | Using where |
+----+-------------+--------+------+-----------------------+------------+---------+-------+------+-------------+
```

MySQL 优化器如果发现可以使用多个索引查找后的交集/并集定位数据，那么 MySQL 优化器就会尝试使用 index merge（索引合并）的方式来查询：

```
EXPLAIN SELECT * FROM user WHERE name = 'admin' AND mail = 'admin@163.com'
```

结果如下：
```
+----+-------------+--------+-------------+-----------------------+-----------------------+---------+------+------+------------------------------------------------------------------+
| id | select_type | table  | type        | possible_keys         | key                   | key_len | ref  | rows | Extra                                                            |
+----+-------------+--------+-------------+-----------------------+-----------------------+---------+------+------+------------------------------------------------------------------+
|  1 | SIMPLE      | user | index_merge | index_name,index_mail | index_name,index_mail | 768,768 | NULL |    1 | Using intersect(index_name,index_mail); Using where; Using index |
+----+-------------+--------+-------------+-----------------------+-----------------------+---------+------+------+------------------------------------------------------------------+
```

对于多列索引，由于索引文件以B树的数据结构存储，MySQL 能够快速转到合适的 name，然后再转到合适的 mail。在建立多列索引时，应该将严格的索引放在前面，这样筛选数据的时候力度会更大，效率更高。


使用分析器分析查询 SQL：
```
EXPLAIN SELECT * FROM user WHERE name = 'admin' AND mail = 'admin@163.com'
```

结果如下：

```
+----+-------------+--------+------+-----------------+-----------------+---------+-------------+------+--------------------------+
| id | select_type | table  | type | possible_keys   | key             | key_len | ref         | rows | Extra                    |
+----+-------------+--------+------+-----------------+-----------------+---------+-------------+------+--------------------------+
|  1 | SIMPLE      | user | ref  | index_name_mail | index_name_mail | 1536    | const,const |    2 | Using where; Using index |
+----+-------------+--------+------+-----------------+-----------------+---------+-------------+------+--------------------------+
```