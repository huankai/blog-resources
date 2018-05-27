---
title: API改进_Stream API
date: {{ date }}
author: huangkai
tags:
    - JDK9
---

Java 的Stream API 是 JDK 8 中添加的新功能，可以让开发者更快的运算，有效的利用数据进行并行计算，能够利用多核架构进行声明式的数据处理。

在JDK9 中，Stream API 变得更加友好：
## 1、Stream 接口中新添加 4 个方法： ##
takeWhile 、dropWhile 、ofNullable 、iterate 重载方法，可以让你提供一个Predicate(判断条件)让你决定什么时候停止循环。

- takeWhile

takeWhile用于从Stream中获取部分数据，接收一个Predicate进行选择，在有序的Stream列表中，takeWile返回从头开始尽量多的元素

```
 Stream<Integer> stream = Stream.of(2, 34, 56, 23, 12, 44, 63, 745);
//从头开始，只要迭代的元素小于50就不再迭代，返回已迭代的元素 // 如只返回  2 和 34
stream.takeWhile(x -> x < 50).forEach(System.out::println);
```

- dropWhile

dorpWhile 和 takeWhile刚好相反

```
Stream<Integer> stream = Stream.of(2, 34, 56, 23, 12, 44, 63, 745);
//和 takeWhile相反，从头开始，只要迭代的元素小于50都不要，返回未迭代的元素 // 如只返回 56, 23, 12, 44, 63, 745
stream.dropWhile(x -> x < 50).forEach(System.out::println);

```

- ofNullable 

```
//对于只有单个元素的 Stream，如果这个元素为 null，会抛出 NullPointerException.
//Stream<Object> stream = Stream.of(null);
//System.out.println(stream.count());

// 如果有多个元素，即使存在空元素，也不会抛出 NullPointerException
Stream.of(3, 5, null).forEach(System.out::println);

System.out.println();

//如果只有单个元素，可以使用 ofNullable 方法
Stream<Object> stream = Stream.ofNullable(null);
System.out.println(stream.count());

```

- iterate 

```
//在jdk 8 中，使用 iterate 方法创建的Stream ，要使用 Limit 方法限制产生的元素个数
Stream.iterate(0, x -> x + 1).limit(10).forEach(System.out::println);

System.out.println();

// 在JDK 9 中，可以使用iterate 的重载方法，接受一个 Predicate 参数，达到指定条件就不再产生元素。
Stream.iterate(0, x -> x < 10, x -> x + 1).forEach(System.out::println);

```

## 2、Optional 类中添加转换为 Stream 的方法 ##

```
List<String> list = new ArrayList<>();
list.add("MM");
list.add("JJ");
list.add("GG");
Optional<List<String>> optionalStrings = Optional.of(list);

//只有一个元素，这个元素是 list
optionalStrings.ifPresent(System.out::println);

// 如果要获取 list 中的每个元素，可以使用 flatMap
Stream<String> stringStream = optionalStrings.stream().flatMap(Collection::stream);
stringStream.forEach(System.out::println);
```