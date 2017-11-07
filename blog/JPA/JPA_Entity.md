---
title: JPA @Entity 注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

@Entity 是类级别注解，用于声明标注的类是持久的，我们把这样的类称为实体类。每个实体类映射到数据库中的一张表，实体类所拥有的属性将映射成数据库表的列。并由 JPA 负责将对实体的操作转换为对数据库表的操作。

| 参数|<font center="true">类型</font>|<font center="true">描述</font>|
| :-------------: |:------------: | :-------------:|
| name     | String 		  | 实体名称，在 JPQL 中用于引用该实体类。默认为该类的简单类名称 |

** 示例 **
```
@Entity(name = "person")
public class Person implements Serializable {
    
    ... ...
    
}
```


