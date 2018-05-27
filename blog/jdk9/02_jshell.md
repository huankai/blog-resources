---
title: java REPL 工具(jshell)
date: {{ date }}
author: huangkai
tags:
    - JDK9
---

# 一、产生背景 #
像 Pyton 、Scala 之类的语言很早就有了交互式编程环境REPL(Read evaluate print loop) ，以交互式的方式对语句和表达式求值，开发者只需要输入一些代码，就可以的编译前获取对程序的反馈。而之前的java版本，想要执行代码，必须先创建文件、声明类、提供测试方法才能实现。

# 二、设计理念 #

<font color='red'>**即写即得，快速运行**</font>

# 三、使用jshell #

 ${JAVA_HOME}/bin 目录下有个 jshell.exe的运行程序，也可以在终端直接运行 jshell 进入java的shell 模式：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/jdk9/jshell_01.jpg)

- 使用jshell 声明变量并运行

```
jshell> int i = 10
i ==> 10

jshell> int j = 20
j ==> 20

jshell> System.out.println(i + j);
30
```

- 使用 jshell声明方法并调用

```
jshell> public void add(int i ,int j){
   ...> System.out.println(i+j);
   ...> }
|  已创建 方法 add(int,int)

jshell> add(1,20)
21

```
注意：如果再次声明相同的方法，会覆盖前面的方法.

- 查看默认导入的包
如果使用以下这些包中的类，可以不使用 import 导入。

```
jshell> /import
|    import java.io.*
|    import java.math.*
|    import java.net.*
|    import java.nio.file.*
|    import java.util.*
|    import java.util.concurrent.*
|    import java.util.function.*
|    import java.util.prefs.*
|    import java.util.regex.*
|    import java.util.stream.*

jshell> 
```

- 查看声明的变量

```
jshell> /vars
|    int i = 10
|    int j = 20

jshell>
```

- 查看声明的方法

```
jshell> /methods 
|    void add(int,int)

jshell>
```

- 执行外部文件

假设 `/Users/huangkai/HelloTest.java`文件内容为:

```
void printHello(){
    System.out.println("Hello world!");
}
printHello();
```
使用 jshell 执行：
```
jshell> /open /Users/huangkai/HelloTest.java
Hello world!

jshell>
```

- 使用文本编辑器:
如要编辑 add方法，

```
jshell> /edit add
#会打开编辑器，可以编辑add方法内容
```

- 没有非RuntimeException 的检查

```
jshell> URL url = new URL("https://www.baidu.com");
url ===> https://www.baidu.com
```

以上这句话，会抛出MalformedURLException，但在 jshell环境不需要throws 或 try catch，内部隐藏实现了。

- 退出jshell

```
jshell> /exit
```