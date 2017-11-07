---
title: JPA @OneToOne   注解
date: {{ date }}
author: huangkai
tags: 
	- JPA
---
@OneToOne 是属性或方法级别的注解，用于定义源实体与目标实体是一对一的关系。

|参数|类型|描述|
|:---:|:---:|---|
|targetEntity|Class|源实体关联的目标实体类型，默认是该成员属性对应的类型，因此该参数通常可以缺省|
|mappedBy|String|用在双向关联中。如果关系是双向的，只能有一方作为主体端，另一方则需声明此参数以表明将表间的这种关联关系转交给对方来维护|
|cascade|CascadeType[]|定义源实体和关联的目标实体间的级联关系。当对源实体进行操作时，是否对关联的目标实体也做相同的操作。默认没有级联操作。该参数的可选值有：CascadeType.PERSIST（级联新建）CascadeType.REMOVE（级联删除）CascadeType.REFRESH（级联刷新）CascadeType.MERGE（级联更新）CascadeType.ALL（包含以上四项）|
|fetch|FetchType|定义关联的目标实体的数据的加载方式。可选值：<br/>FetchType.LAZY（延迟加载）<br/>FetchType.EAGER（立即加载，默认）<br/>延迟加载：只有在第一次访问源实体关联的目标实体的时候才去加载。<br/>立即加载：在加载源实体数据的时候同时去加载好关联的目标实体的数据|
|optional|boolean|源实体关联的目标实体是否允许为 null，默认为 true|
|orphanRemoval|boolean|当源实体关联的目标实体被断开（如给该属性赋予另外一个实例，或该属性的值被设为 null。被断开的实例称为孤值，因为已经找不到任何一个实例与之发生关联）时，是否自动删除断开的实例（在数据库中表现为删除表示该实例的行记录），默认为 false|

注：目标实体是指被关系注解（如：@OneToOne）标注的属性或方法所对应的类，源实体是指该属性或方法所属的类。

# 1. 一对一单向外键关联 #

```
@Entity(name = "person")
public class Person implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private IdCard idCard;
    
    // getters and setters
    
}
```

```
@Entity(name = "idcard")
public class IdCard {
    
    @Id
    private String no;
    
    @Temporal(TemporalType.DATE)
    private Date expiryDate;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：

```
CREATE TABLE `person` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `id_card_no` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FKar03p8ob32rgj1axxy2q507v5` (`id_card_no`),
  CONSTRAINT `FKar03p8ob32rgj1axxy2q507v5` FOREIGN KEY (`id_card_no`) REFERENCES `idcard` (`no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
    
CREATE TABLE `idcard` (
  `no` varchar(255) NOT NULL,
  `expiry_date` date DEFAULT NULL,
  PRIMARY KEY (`no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

# 2. orphanRemoval 与 CascadeType.REMOVE 的区别 #

CascadeType.REMOVE（或包含 CascadeType.REMOVE 的 CascadeType.ALL）表示级联删除，只有对源实例做删除操作时，才会级联删除关联的目标实例。如上例，删除 id=1 的 Person 实例，那么，该实例所关联的 IdCard 实例也将被删除。示例代码片段：

```
personRepository.delete(1L);
```

假设 person.idCard = idCard1，如果 person.idCard 属性被赋予了另外一个 IdCard 实例：person.idCard = idCard2 或被设为 null：person.idCard = null。此时，身份证 idCard1 已经找不到任何一个人和它发生关联，这样的值我们成为孤值，它已经失去了存在的意义。如果程序中设置了 orphanRemoval = true，那么，当更新 person 实例时，idCard1 实例将会被自动删除。示例代码片段：

```
Person person = personRepository.findOne(1L);
person.setIdCard(null);
personRepository.save(person);
```

# 3. 一对一双向外键关联 #
类 Person 定义不变，IdCard 类的定义改为：

```
@Entity(name = "idcard")
public class IdCard {
    
    @Id
    private String no;
    
    @Temporal(TemporalType.DATE)
    private Date expiryDate;
    
    @OneToOne(mappedBy = "idCard")
    private Person person;
    
    // getters and setters
    
}
```
产生的 DDL 语句与一对一单向外键关联产生的一致。


# 4. @JoinColumn #

|参数|类型|描述|
|:---:|:---:|---|
|name|String|外键列的名称，默认为：属性的名称 + \_ + 属性对应的实体的主键列的名称（Hibernate 映射列时，若遇到驼峰拼写，会自动添加 \_ 连接并将大写字母改成小写）。
|unique|boolean|外键列的值是否是唯一的。这是 @UniqueConstraint 注解的一个快捷方式，实质上是在声明唯一约束。默认值为 false|
|nullable|boolean|外键列的值是否允许为 null。默认为 true|
|insertable|boolean|外键列是否包含在 INSERT 语句中，默认为 true|
|updatable|boolean|外键列是否包含在 UPDATE 语句中，默认为 true|
|columnDefinition|String|生成外键列的 DDL 时使用的 SQL 片段。默认使用推断的类型来生成 SQL 片段以创建此列|
|table|String|外键列所属的表的名称。默认值：<br/>如果是外键 @OneToOne 或 @ManyToOne 关联，则为源实体的表的名称；<br/>如果是单向外键 @OneToMany 关系，则为目标实体的表的名称；<br/>如果是 @ManyToMany、@OneToOne、双向 @ManyToOne、双向 @OneToMany 关联，则为连接表的名称|

若修改 Person 类的定义为：
```
@Entity(name = "person")
public class Person implements Serializable {
    
    @Id
    @GeneratedValue
    private Long id;
    
    private String name;
    
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "idcard_no")
    private IdCard idCard;
    
    // getters and setters
    
}
```

产生的 DDL 语句（MySQL）：
```
CREATE TABLE `person` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `idcard_no` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FK56rk440ff4uyhc5vdis0jeiut` (`idcard_no`),
  CONSTRAINT `FK56rk440ff4uyhc5vdis0jeiut` FOREIGN KEY (`idcard_no`) REFERENCES `idcard` (`no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```