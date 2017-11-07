---
title: JPA @Id   注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

@Id 是属性或方法级别的注解，该注解没有参数，用于标注实体的主键（映射到数据库表的主键）。

## 示例： ##

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    private Long id;
    
    private String name;
    
    private String mail;
    
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
```