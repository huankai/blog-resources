---
title: JPA @Temporal   注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

@Temporal 是属性或方法级别的注解，用于声明属性持久化到数据库时所使用的时间精度。该注解可以应用于任何以下类型的实体类属性：
- java.util.Date
- java.util.Calendar


|参数|类型|描述|
|:---:|:---:|---|
|value|TemporalType|存储的类型，可选值：<br/>TemporalType.DATE（日期）<br/>TemporalType.TIME（时间）<br/>TemporalType.TIMESTAMP（日期和时间）|

## 示例： ##

```
@Entity(name = "user")
public class User implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    @Temporal(TemporalType.DATE)
    private Date birthday;
    
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastLoginTime;
    
    @Temporal(TemporalType.TIME)
    private Date tokenExpiredTime;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `birthday` date DEFAULT NULL,
  `last_login_time` datetime DEFAULT NULL,
  `token_expired_time` time DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

数据值样例：
```
+----+------------+---------------------+--------------------+
| id | birthday   | last_login_time     | token_expired_time |
+----+------------+---------------------+--------------------+
|  1 | 2017-05-14 | 2017-05-14 15:12:49 | 15:12:49           |
+----+------------+---------------------+--------------------+
```