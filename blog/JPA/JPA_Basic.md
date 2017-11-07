---
title: JPA @Basic   注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

@Basic 是属性或方法级别的注解，该注解可以应用于任何以下类型的实体类属性：
- Java 原始类型
- 原始类型的包装类型
- String
- java.math.BigInteger
- java.math.BigDecimal
- java.util.Date
- java.util.Calendar
- java.sql.Date
- java.sql.Time
- java.sql.Timestamp
- byte[]
- Byte[]
- char[]
- Character[]
- 枚举
- 任意实现 java.io.Serializable 接口的类型

在实体类中，对以上这些类型的属性，如果没有标注 @Basic 注解，则将使用 @Basic 注解的默认值

|参数|类型|描述|
|:---:|:---:|---|
|fetch|FetchType|属性值的加载策略。可选值：<br/>FetchType.EAGER：即时加载；<br/>FetchType.LAZY：延迟加载，当第一次访问属性时才进行数据的加载；<br/>默认为 FetchType.EAGER。|
|optional|boolean|是否允许为 null，默认为 true|

## 示例： ##

```
@Entity(name = "student")
public class Student implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    private String name;
    
    @Basic(optional = false, fetch = FetchType.LAZY)
    private String mail;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `student` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `mail` varchar(255) NOT NULL,
  `name` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```