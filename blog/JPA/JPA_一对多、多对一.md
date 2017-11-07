---
title: JPA 一对多与多对一
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

# 1、OneToMany #

@OneToMany 是属性或方法级别的注解，用于定义源实体与目标实体是一对多的关系。

|参数|类型|描述|
|:---:|:---:|---|
|targetEntity|Class|源实体关联的目标实体类型，默认是该成员属性对应的集合类型的泛型的参数化类型|
|mappedBy|String|用在双向关联中。如果关系是双向的，则需定义此参数（与 @JoinColumn 互斥，如果标注了 @JoinColumn 注解，不需要再定义此参数）|
|cascade|CascadeType[]|定义源实体和关联的目标实体间的级联关系。当对源实体进行操作时，是否对关联的目标实体也做相同的操作。默认没有级联操作。该参数的可选值有：CascadeType.PERSIST（级联新建）CascadeType.REMOVE（级联删除）CascadeType.REFRESH（级联刷新）CascadeType.MERGE（级联更新）CascadeType.ALL（包含以上四项）|
|fetch|FetchType|定义关联的目标实体的数据的加载方式。可选值：<br/>FetchType.LAZY（延迟加载，默认）<br/>FetchType.EAGER（立即加载）|
|orphanRemoval|boolean|当源实体关联的目标实体被断开（如给该属性赋予另外一个实例，或该属性的值被设为 null。被断开的实例称为孤值，因为已经找不到任何一个实例与之发生关联）时，是否自动删除断开的实例（在数据库中表现为删除表示该实例的行记录），默认为 false|

## 1.1、 一对多外键关联 ##
```
@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String username;
    
    private String password;
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Address> addresses;
    
    // getters and setters
    
}
```


```
@Entity(name = "address")
public class Address implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    private String province;
    
    private String city;
    
    private String area;
    
    private String detail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `password` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `address` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `area` varchar(255) DEFAULT NULL,
  `city` varchar(255) DEFAULT NULL,
  `detail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `province` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `user_addresses` (
  `user_id` bigint(20) NOT NULL,
  `addresses_id` bigint(20) NOT NULL,
  PRIMARY KEY (`user_id`,`addresses_id`),
  UNIQUE KEY `UK_i5lp1fvgfvsplfqwu4ovwpnxs` (`addresses_id`),
  CONSTRAINT `FKfm6x520mag23hvgr1oshaut8b` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`),
  CONSTRAINT `FKth1icmttmhhorb9wiarm73i06` FOREIGN KEY (`addresses_id`) REFERENCES `address` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

Hibernate @OneToMany 默认会产生一张中间表，如上例的 user_addresses 表。为了避免这种情况，你可以在一的一方使用 @JoinColumn 注解：

```
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
@JoinColumn(name = "user_id")
private Set<Address> addresses;
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `password` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `address` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `area` varchar(255) DEFAULT NULL,
  `city` varchar(255) DEFAULT NULL,
  `detail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `province` varchar(255) DEFAULT NULL,
  `user_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FKda8tuywtf0gb6sedwk7la1pgi` (`user_id`),
  CONSTRAINT `FKda8tuywtf0gb6sedwk7la1pgi` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
这样一来，多的一方通过外键直接与一的一方发生关联，不需要中间表。

# 2、@ManyToOne #
@ManyToOne 是属性或方法级别的注解，用于定义源实体与目标实体是多对一的关系，属性参数见 @OneToMany 。

## 2.1、 多对一外键关联 ##

```

@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String username;
    
    private String password;
    
    // getters and setters
    
}
```

```
@Entity(name = "address")
public class Address implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    private String province;
    
    private String city;
    
    private String area;
    
    private String detail;
    
    @ManyToOne(optional = false)
    private User user;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `password` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `address` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `area` varchar(255) DEFAULT NULL,
  `city` varchar(255) DEFAULT NULL,
  `detail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `province` varchar(255) DEFAULT NULL,
  `user_id` bigint(20) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `FKda8tuywtf0gb6sedwk7la1pgi` (`user_id`),
  CONSTRAINT `FKda8tuywtf0gb6sedwk7la1pgi` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

# 3、 @OneToMany & @ManyToOne #

一对多 & 多对一双向外键关联示例：

```
@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String username;
    
    private String password;
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "user_id")
    private Set<Address> addresses;
    
    // getters and setters
    
}
```

```
@Entity(name = "address")
public class Address implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    private String province;
    
    private String city;
    
    private String area;
    
    private String detail;
    
    @ManyToOne(optional = false)
    private User user;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `password` varchar(255) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `address` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `area` varchar(255) DEFAULT NULL,
  `city` varchar(255) DEFAULT NULL,
  `detail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  `province` varchar(255) DEFAULT NULL,
  `user_id` bigint(20) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `FKda8tuywtf0gb6sedwk7la1pgi` (`user_id`),
  CONSTRAINT `FKda8tuywtf0gb6sedwk7la1pgi` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```