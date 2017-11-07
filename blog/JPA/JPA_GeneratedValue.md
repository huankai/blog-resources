---
title: JPA @GeneratedValue  注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

@GeneratedValue 是属性或方法级别的注解，它结合 @Id 注解为主键的值提供生成策略的规范。

|参数|类型|描述|
|:---:|:---:|---|
|strategy|GenerationType|主键生成策略。可选值：<br/>GenerationType.TABLE<br/>GenerationType.SEQUENCE<br/>GenerationType.IDENTITY<br/>GenerationType.AUTO <br/>默认是 GenerationType.AUTO|
|generator|String|主键生成器的名称。该名称为 @TableGenerator 或 @SequenceGenerator 注解中 name 参数的值。默认为持久化提供者（如 Hibernate）提供的id生成器|

# 1、 @GeneratedValue #

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `student` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 1.1、 GenerationType.TABLE ##
该策略使用一个特殊的数据库表来为各个实体分配主键，并确保主键值的唯一性。它不依赖环境和数据库系统的具体实现，在不同数据库之间可以很容易的进行移植。

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

在 Hibernate + MySQL 环境产生的 DDL 语句：

```
CREATE TABLE `student` (
  `id` bigint(20) NOT NULL,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `hibernate_sequences` (
  `sequence_name` varchar(255) NOT NULL,
  `sequence_next_hi_value` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`sequence_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
其中，hibernate_sequences 表就是用来为各个实体分配主键的，sequence_name 用于存储各个实体名称，sequence_next_hi_value 则是用于存储各个实体的下一个主键的值。

每次保存 Student 实体对象的数据的时候，首先会到 hibernate_sequences 表中查询该实体名称是否已经存在，若不存在，则向 hibernate_sequences 表插入一行该实体名称的记录，若已经存在，则直接取出 sequence_next_hi_value 的值作为本次 Student 数据的主键值，接着更新 hibernate_sequences 表的记录，使该实体名称的记录的 sequence_next_hi_value 值加1。最后保存 Student 实体对象的记录。

## 1.1.1、 @TableGenerator ###

表生成器，GenerationType.TABLE 策略通常结合该注解一起使用。

|参数|类型|描述|
|:---:|:---:|---|
|name|String|生成器的名称，它可以被一个或多个类引用为主键的生成器|
|table|String|生成器表的名称。默认为持久化提供者（如 Hibernate）提供的名称|
|catalog|String|生成器表的 catalog，默认为数据库系统缺省的 catalog|
|schema|String|生成器表的 schema，默认为用户缺省的 schema|
|pkColumnName|String|生成器表的主键列的名称，默认为持久化提供者（如 Hibernate）提供的名称|
|valueColumnName|String|生成器表存储生成的值的列的名称，默认为持久化提供者（如 Hibernate）提供的名称|
|pkColumnValue|String|生成器表中的主键值，用于将不同的实体区分开来。默认为实体名称（@Entity 注解的 name 参数的值）|
|initialValue|int|用于初始化生成器生成的初始值。默认值是0。在 Hibernate 环境中，需要开启 hibernate.id.new_generator_mappings=true（Spring JPA 配置为：spring.jpa.hibernate.use-new-id-generator-mappings=true），否则此参数无效。并且如果实体记录已经存在生成器表中，此参数也无效（即只有当实体记录第一次写入生成器表中时此参数生效）|
|allocationSize|int|每次分配的值的数量大小，用完之后再分配此数量的值，默认为50|
|uniqueConstraints|UniqueConstraint[]|表的唯一约束|
|indexes|Index[]|表的索引|

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "pkGenerator")
    @TableGenerator(
            name = "pkGenerator",
            table = "pk_sequences",
            pkColumnName = "entity_name",
            valueColumnName = "sequence_value",
            allocationSize = 10
    )
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

```
@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, generator = "pkGenerator")
    @TableGenerator(
            name = "pkGenerator",
            table = "pk_sequences",
            pkColumnName = "entity_name",
            valueColumnName = "sequence_value",
            allocationSize = 10
    )
    private Long id;
    
    private String name;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `student` (
  `id` bigint(20) NOT NULL,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `pk_sequences` (
  `entity_name` varchar(255) NOT NULL,
  `sequence_value` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`entity_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 1.2、 GenerationType.SEQUENCE ##
某些数据库系统（如：Oracle、DB2、PostgreSQL 等）底层支持使用序列对象来为表的主键提供唯一值，而对于不支持序列对象的数据库（如：MySQL、SQLServer），则不应使用此策略。

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

```
@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;
    
    private String name;
    
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
}
```
这种方式是多个实体共用同一个序列对象，这将导致各个实体分配到的序列值不连续，并且消耗加快

### 1.2.1、 @SequenceGenerator ###
序列生成器，GenerationType.SEQUENCE 策略通常结合该注解一起使用。

|参数|类型|描述|
|:---:|:---:|---|
|name|String|生成器的名称，它可以被一个或多个类引用为主键的生成器|
|sequenceName|String|序列对象的名称，默认为持久化提供者（如 Hibernate）提供的名称|
|catalog|String|序列生成器的 catalog|
|schema|String|序列生成器的 schema|
|initialValue|int|用于初始化序列对象生成的初始值。默认值是1|
|allocationSize|int|每次分配的值的数量大小，用完之后再分配此数量的值，默认为50|

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "personSeqGenerator")
    @SequenceGenerator(
            name = "personSeqGenerator",
            sequenceName = "PERSON_SEQ",
            allocationSize = 10
    )
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

```
@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "userSeqGenerator")
    @SequenceGenerator(
            name = "userSeqGenerator",
            sequenceName = "USER_SEQ",
            allocationSize = 10
    )
    private Long id;
    
    private String name;
    
    // getters and setters
    
}
```

## 1.3、 GenerationType.IDENTITY ##

大部分数据库系统（如：MySQL、SQLServer、DB2、Sybase、HypersonicSQL 等）底层支持表的主键自增长，而对于不支持主键自增长的数据库（如：Oracle），则不能使用此策略。

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `person` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 1.4、 GenerationType.AUTO ##
自动为特定的数据库选择适当的策略，这是比较常用的策略。

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```
由于 @GeneratedValue 注解默认采用的策略就是 GenerationType.AUTO，因此可以简写成：
```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```



