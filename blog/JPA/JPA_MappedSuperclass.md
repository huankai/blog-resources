---
title: JPA @MappedSuperclass  注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

@MappedSuperclass 是类级别注解，该注解没有任何参数，被该注解标注的类不会映射到数据库中单独的表，但该类所拥有的属性都将映射到其子类的数据库表的列中。

## 1、 示例 ##

```
@MappedSuperclass
public class BaseEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    protected Long id;
    
    @Column(name = "create_time")
    protected Date createTime;
    
    // getters and setters
    
}
```

```
@Entity(name = "student")
public class Student extends BaseEntity implements Serializable {
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `student` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `create_time` datetime DEFAULT NULL,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 2、 @AttributeOverride ##
对于被 @MappedSuperclass 注解标注的类派生出来的子类，可以使用 @AttributeOverride 注解重新定义以覆盖父类中的映射信息。

|参数|类型|描述|
|:---:|:---:|:---:|
|name|String|属性名称|
|column|Column|列信息|

```
@Entity(name = "student")
@AttributeOverride(name = "createTime", column = @Column(name = "create_date"))
public class Student extends BaseEntity implements Serializable {
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `student` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `create_date` datetime DEFAULT NULL,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 3、 @AttributeOverrides ##
如果需要重新定义父类中的多个映射信息，需要使用 @AttributeOverrides 注解。

|参数|类型|描述|
|:---:|:---:|:---:|
|value|AttributeOverride[]|@AttributeOverride 注解列表|

```
@Entity(name = "student")
@AttributeOverrides({
        @AttributeOverride(name = "id", column = @Column(name = "student_id")),
        @AttributeOverride(name = "createTime", column = @Column(name = "create_date"))
})
public class Student extends BaseEntity implements Serializable {
    
    private String name;
    
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `student` (
  `student_id` bigint(20) NOT NULL AUTO_INCREMENT,
  `create_date` datetime DEFAULT NULL,
  `mail` varchar(255) DEFAULT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`person_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```