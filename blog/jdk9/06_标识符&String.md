---
title: 语法改进_标识符&String
date: {{ date }}
author: huangkai
tags:
    - JDK9
---

# 一、标识符： #

在 jdk8之前 ，可以使用 下划线声明变量，在jdk9 中，单独的下划线有特殊的含义，不能被用于标识符。

```
String _ = "test"; //在jdk8 之前编译通过，jdk9中编译不通过。
```

# 二、String  #

在JDK8之前，String 使用的是 char类型的数组，在JDK9中，改为了byte类型的数组(encoding flag)。

char类型使用的是两个字节，在多数情况下，String类型在堆存储时大多数是Latin字符，而Latin字符只需要一个byte就可以，如果使用 char类型，就浪费了一半的空间。我们知道对于中文，使用是三个字节，但是在UTF-16中，只暂用两个字节。