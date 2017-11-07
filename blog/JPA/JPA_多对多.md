---
title: JPA 多对多
date: {{ date }}
author: huangkai
tags: 
	- JPA
---


@ManyToMany 是属性或方法级别的注解，用于定义源实体与目标实体是多对多的关系。

|参数|类型|描述|
|:---:|:---:|---|
|targetEntity|Class|源实体关联的目标实体类型，默认是该成员属性对应的集合类型的泛型的参数化类型|
|mappedBy|String|用在双向关联中。如果关系是双向的，则需定义此参数（与 @JoinColumn 互斥，如果标注了 @JoinColumn 注解，不需要再定义此参数）|
|cascade|CascadeType[]|定义源实体和关联的目标实体间的级联关系。当对源实体进行操作时，是否对关联的目标实体也做相同的操作。默认没有级联操作。该参数的可选值有：CascadeType.PERSIST（级联新建）CascadeType.REMOVE（级联删除）CascadeType.REFRESH（级联刷新）CascadeType.MERGE（级联更新）CascadeType.ALL（包含以上四项）|
|fetch|FetchType|定义关联的目标实体的数据的加载方式。<br/>可选值：<br/>FetchType.LAZY（延迟加载，默认）<br/>FetchType.EAGER（立即加载）|

# 1. 多对多单向外键关联 #
```
Entity(name = "course")
public class Course {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    // getters and setters
    
}
```

```
@Entity(name = "student")
public class Student {
    
    @Id
    private String no;
    
    private String name;
    
    @ManyToMany(cascade = CascadeType.ALL)
    private Set<Course> courses;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `course` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `student` (
  `no` varchar(255) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `student_courses` (
  `student_no` varchar(255) NOT NULL,
  `courses_id` bigint(20) NOT NULL,
  PRIMARY KEY (`student_no`,`courses_id`),
  KEY `FKlwviiijdg10oc2ui4yl7adh1o` (`courses_id`),
  CONSTRAINT `FKa6x7sxxnd9c1pat349a01bsow` FOREIGN KEY (`student_no`) REFERENCES `student` (`no`),
  CONSTRAINT `FKlwviiijdg10oc2ui4yl7adh1o` FOREIGN KEY (`courses_id`) REFERENCES `course` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

# 2. @JoinTable #

与 @Table 注解相类似，不同的是，@JoinTable 注解是用于定义关联表，它只能标注在实体类型的成员属性或方法上，常用于多对多或多对一的关联映射。如果没有声明，则使用该注解的默认值。

|参数|类型|描述|
|:---:|:---:|---|
|name|String|连接表的名称|
|catalog|String|默认为数据库系统缺省的 catalog|
|schema|String|默认为用户缺省的 schema|
|joinColumns|JoinColumn[]|连接表中的外键列，通过使用 @JoinColumn 注解来声明，该外键参照源实体的主键|
|inverseJoinColumns|JoinColumn[]|与 joinColumns 参数作用类似，只不过该外键参照的是目标实体的主键|
|uniqueConstraints|UniqueConstraint[]|表的唯一约束（除了由 @Column 和 @JoinColumn 注解指定的约束以及主键的约束之外的约束），通过使用 @UniqueConstraint 注解来声明，仅在允许自动更新数据库表结构的场景中起到作用，默认没有其他额外的约束条件|
|indexes|Index[]|表的索引，通过使用 @Index 注解来声明，仅在允许自动更新数据库表结构的场景中起到作用，默认没有其他额外的索引|
|foreignKey|ForeignKey|用于生成表时定义 joinColumns 参数的外键约束|
|inverseForeignKey|ForeignKey|用于生成表时定义 inverseJoinColumns 参数的外键约束|

Course 定义不变，Student 定义改为：
```
@Entity(name = "student")
public class Student {
    
    @Id
    private String no;
    
    private String name;
    
    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "student_course",
            joinColumns = @JoinColumn(name = "sno"),
            inverseJoinColumns = @JoinColumn(name = "cid"))
    private Set<Course> courses;
    
    // getters and setters
    
}
```
产生的 DDL 语句（MySQL）：
```
CREATE TABLE `course` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `student` (
  `no` varchar(255) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `student_course` (
  `sno` varchar(255) NOT NULL,
  `cid` bigint(20) NOT NULL,
  PRIMARY KEY (`sno`,`cid`),
  KEY `FKkx4bkddvbfs0ese9v7hc5rycg` (`cid`),
  CONSTRAINT `FKrfibef5g98fllv2tlxuiii0lu` FOREIGN KEY (`sno`) REFERENCES `student` (`no`),
  CONSTRAINT `FKkx4bkddvbfs0ese9v7hc5rycg` FOREIGN KEY (`cid`) REFERENCES `course` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

# 3. 多对多双向外键关联 #

类 Student 定义不变，Course 类的定义改为：

```
@Entity(name = "course")
public class Course {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students;
    
    // getters and setters
    
}
```

产生的 DDL 语句与多对多单向外键关联产生的一致。



