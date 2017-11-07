---
title: JPA @Column   注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---


@Column 是属性或方法级别的注解，用于指定持久化属性映射到数据库表的列。如果没有指定列注释，则使用其默认值。

|参数|类型|描述|
|:---:|:---:|---|
|name|String|列的名称，默认为属性的名称（Hibernate 映射列时，若遇到驼峰拼写，会自动添加 _ 连接并将大写字母改成小写）|
|unique|boolean|列的值是否是唯一的。这是 @UniqueConstraint 注解的一个快捷方式， 实质上是在声明唯一约束。默认值为 false|
|nullable|boolean|列的值是否允许为 null。默认为 true|
|insertable|boolean|列是否包含在 INSERT 语句中，默认为 true|
|updatable|boolean|列是否包含在 UPDATE 语句中，默认为 true|
|columnDefinition|String|生成列的 DDL 时使用的 SQL 片段。默认使用推断的类型来生成 SQL 片段以创建此列|
|table|String|当前列所属的表的名称|
|length|int|列的长度，仅对字符串类型的列生效。默认为255|
|precision|int|列的精度，仅对十进制数值有效，表示有效数值的总位数。默认为0|
|scale|int|列的精度，仅对十进制数值有效，表示小数位的总位数。默认为0|

# 1、 示例 #
```
@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @Column(nullable = false, length = 32)
    private String name;
    
    @Column(length = 128)
    private String mail;
    
    @Column(columnDefinition = "char(11) NOT NULL")
    private String phone;
    
    @Column(precision = 5, scale = 2)
    private BigDecimal salary;
    
    @Column(precision = 5, scale = 2)
    private double assets;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `assets` double NOT NULL,
  `mail` varchar(128) DEFAULT NULL,
  `name` varchar(32) NOT NULL,
  `phone` char(11) NOT NULL,
  `salary` decimal(5,2) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

可以看出，salary 字段的精度控制生效了，但对于 double 类型的 assets 字段的精度控制没有生效，为了使其生效，将代码修改为：
```
@Column(columnDefinition = "double(5, 2)")
private double assets;
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `assets` double(5,2) DEFAULT NULL,
  `mail` varchar(128) DEFAULT NULL,
  `name` varchar(32) NOT NULL,
  `phone` char(11) NOT NULL,
  `salary` decimal(5,2) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```