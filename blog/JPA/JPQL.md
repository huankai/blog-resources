---
title: JPQL语法
date: {{ date }}
author: huangkai
tags: 
	- JPA
---

JPQL（Java Persistence Query Language，Java 持久化查询语言）和 SQL 之间有很多相似之处，它们之间主要的区别在于前者处理 JPA 实体类，而后者则直接涉及关系数据。在 JPQL 中，可以使用SELECT、UPDATE和DELETE语法来定义查询。

# 1. 查询 #

语法：``SELECT ... FROM ... [WHERE ...] [GROUP BY ... [HAVING ...]] [ORDER BY ...]``

**FROM 子句** 
通过声明一个或多个标识符变量来定义查询的范围

**WHERE 子句 **
用于限制查询到的对象或值的条件表达式

**GROUP BY 子句**
根据一组属性对查询结果进行分组

**HAVING 子句**
配合GROUP BY子句使用，以根据条件表达式进一步限制查询结果

**ORDER BY 子句**
对查询结果进行排序

```
//部门
@Entity(name = "department")
public class Department {
	
    @Id
    @GeneratedValue
    private Long id;
	
    private String name;
	
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "department_id")
    private Set<Employee> employees;
	
    // getters and setters
	
}

```

```
//员工
@Entity(name = "employee")
public class Employee {
	
    @Id
    @GeneratedValue
    private Long id;
	
    private String name;
	
    @Enumerated(EnumType.STRING)
    private Sex sex;
	
    private Integer age;
	
    private Boolean married;
	
    private Double salary;
	
    private Date hireDate;
	
    @ManyToOne(fetch = FetchType.LAZY)
    private Department department;
	
    // getters and setters
	
}
```

```
public enum Sex {
	
    MALE("男"),
	
    FEMALE("女"),
	
    ;
	
    private final String displayText;
	
    private Sex(String displayText) {
        this.displayText = displayText;
    }
	
    @Override
    public String toString() {
        return displayText;
    }
	
}
```

## 1.1 基础查询 ##
语法：``SELECT 标识符变量 FROM 实体名称 [AS] 标识符变量``
示例：查询所有的雇员信息
```
@Query("SELECT E FROM Employee E")
List<Employee> selectAll();
```

## 1.2 查询参数 ##

JPQL 支持两种查询参数，它们分别是命名参数和位置参数

### 1.2.1 命名参数 ###

语法：``:自定义的参数名称``
示例：按性别和薪资范围查找雇员信息
```
@Query("SELECT E FROM Employee E WHERE E.sex = :sex AND E.salary > :salary")
List<Employee> selectByNamedParams(@Param("sex") Sex sex, @Param("salary") Double salary);
```
在方法的参数列表中，需要使用@Param注解标注每个参数的名称，使之与查询语句参数名称匹配

### 1.2.2 位置参数 ###

语法：``?位置编号的数值``
示例：按姓名和性别查找雇员信息

```
@Query("SELECT E FROM Employee E WHERE E.sex = ?1 AND E.salary > ?2")
List<Employee> selectByPositionalParams(Sex sex, Double salary);
```
在方法的参数列表中，参数的顺序需要与查询语句中参数标注的编号依次对应起来。

## 1.3 关联查询 ##

通过使用关键字[LEFT|INNER] JOIN联接关系属性查询

### 1.3.1 单值关联查询 ###

语法：``SELECT 标识符变量 FROM 实体名称 [AS] 标识符变量 JOIN 实体名称.单值关联字段 [AS] 标识符变量2 ...``

示例：按部门名称查找该部门所有的雇员信息
```
@Query("SELECT E FROM Employee E JOIN E.department D WHERE D.name = ?1")
List<Employee> selectByDeptName(String deptName);
```

### 1.3.2 多值关联查询 ###

语法1：``SELECT 标识符变量 FROM 实体名称 [AS] 标识符变量 JOIN 实体名称.多值关联字段 [AS] 标识符变量2 ...``
示例：查询薪资大于10000的所有雇员所属的部门信息
```
@Query("SELECT D FROM Department D JOIN D.employees E WHERE E.salary > 10000")
List<Department> selectByMultRelatedField();
```

语法2：``SELECT 标识符变量 FROM 实体名称 [AS] 标识符变量, IN(实体名称.多值关联字段) [AS] 标识符变量2 ...``
```
@Query("SELECT D FROM Department D, IN(D.employees) E WHERE E.salary > 10000")
List<Department> selectByMultRelatedCollection();
```

## 1.4 去重查询 ##

语法：``SELECT DISTINCT 标识符变量 FROM 实体名称 [AS] 标识符变量 ...``
示例：查询薪资大于10000的所有雇员所属的部门信息，并消除查询结果中的重复的部门

```
@Query("SELECT DISTINCT D FROM Department D JOIN D.employees E WHERE E.salary > 10000")
List<Department> selectByMultRelatedFieldDistinct();
```

## 1.5 字面值 ##

JPQL 支持的字面值有以下的4种，它们分别是：字符串、数字、布尔、枚举。

### 1.5.1 字符串 ###
语法：``'字符串'``
示例：查询给定名字的雇员信息

```
@Query("SELECT E FROM Employee E WHERE E.name = '张三'")
Employee selectByLiteralString();
```

如果字符串中含有单引号，则用两个单引号来表示。如：Li'Si -> Li''Si

```
@Query("SELECT E FROM Employee E WHERE E.name = 'Li''Si'")
Employee selectByLiteralStringWithQuote();
```

### 1.5.2 数字 ###
整数类型：如``24、+24、-24、24L``，支持 Java Long 范围的数值。
浮点类型：如``24.、24.6、+24.6、-24.6、24.6F、24.6D``，支持 Java Double 范围的数值。
示例：查询薪资大于10000的所有雇员
```
@Query("SELECT E FROM Employee E WHERE E.salary > 10000.0")
List<Employee> selectByLiteralNumber();
```

### 1.5.3 布尔 ###
布尔类型的可选值为：``TRUE``或``FALSE``，它们不区分大小写。
示例：查找已婚的所有雇员

```
@Query("SELECT E FROM Employee E WHERE E.married = TRUE")
List<Employee> selectByLiteralBool();
```

### 1.5.4 枚举 ###
枚举类名必须指定为完全限定类名。
示例：查询所有女性的雇员
```
@Query("SELECT E FROM Employee E WHERE E.sex = org.fanlychie.enums.Sex.FEMALE")
List<Employee> selectByLiteralEnum();
```

## 1.6 模糊查询 ##

|表达式|匹配|不匹配|
|:---:|:---:|---|
|E.name LIKE ‘张%’|张三|小张伟|
|E.name LIKE ‘张_’|张三|张三丰|
|E.name LIKE ‘张\_%|张_三|张三|

示例：查询张性的所有雇员
```
@Query("SELECT E FROM Employee E WHERE E.name LIKE '张%'")
List<Employee> selectByLikeLiteralString();
```

## 1.7 空集合查询 ##

通过使用关键字``IS [NOT] EMPTY``来查找关联的属性集合的值为空的记录。
示例：查找尚无雇员的所有部门
```
@Query("SELECT D FROM Department D WHERE D.employees IS EMPTY")
List<Department> selectByEmpty();
```

## 1.8 构造器 ##

查询结果的类型如果不是持久化的实体类，必须使用该类的完全限定名。
语法：``SELECT NEW`` 类的完全限定名(参数1, 参数2, ...) ...
示例：查询所有的雇员信息
```
@Query("SELECT NEW com.hk.model.SimpleEmployee(E.name, E.sex) FROM Employee E")
List<SimpleEmployee> selectSimpleEmployees();
```

```
package com.hk.model;
	
public class SimpleEmployee {
	
    private String name;
	
    private Sex sex;
	
    public SimpleEmployee(String name, Sex sex) {
        this.name = name;
        this.sex = sex;
    }
	
    // getters and setters
    
}
```

# 2. 更新 #

示例：更新某个雇员的婚姻状态和薪资信息

```
@Modifying
@Transactional
@Query("UPDATE Employee SET married = ?2, salary = ?3 WHERE id = ?1")
int update(Long id, Boolean married, Double salary);
```
@Query无法进行 DML（Data Manipulation Language 数据操控语言，主要语句有 INSERT、DELETE、UPDATE）操作，如需更新数据库表的数据需要标注@Modifying注解，并且需要使用支持事务的@Transactional注解。


# 3. 删除 #

示例：删除没有雇员的部门信息

```
@Modifying
@Transactional
@Query("DELETE FROM Department D WHERE D.employees IS EMPTY")
int delete();
```

参考文档文献链接：[https://docs.oracle.com/javaee/7/tutorial/persistence-querylanguage.htm](https://docs.oracle.com/javaee/7/tutorial/persistence-querylanguage.htm) 、[https://docs.oracle.com/html/E13946_04/ejb3_langref.html](https://docs.oracle.com/html/E13946_04/ejb3_langref.html)





