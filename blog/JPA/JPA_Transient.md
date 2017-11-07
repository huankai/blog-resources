---
title: JPA @Transient  注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---
@Transient 是属性或方法级别的注解，该注解没有参数，用于标注属性是瞬态而非持久的。


## 示例: ##
```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @Transient
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `student` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
