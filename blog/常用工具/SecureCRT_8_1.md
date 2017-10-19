---
title: SecureCRT 8.1 安装与破解
date: {{ date }}
author: huangkai
tags:
categories:
    - 常用工具
---

# 一、安装 #
下载地址： [百度云盘链接](https://pan.baidu.com/s/1o8iLaII) ，密码 ： **dfit**

安装直接按步骤下一步完成即可，这里略过

# 二、破解 #
先不要启动程序，将 keygen.exe与 fx.exe文件复制到安装目录下
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/01.png)

右键 keygen.exe 以管理员运行如下 ：
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/02.png)

点击 patch,弹出对话框，选中 SecureCRT.exe 文件，再点击 Generate，选择 LicenseHelper.exe 文件，破解 SecureCRT。

右键 fx.exe 以管理员运行如下 ：
![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/03.png)
```
Name 输入上面的 name,如： ygeR
Company 输入上面的 Company,如： TEAM ZWT
点击Patch --> Generate
```
然后打开SecureCRT 和 SecureFX，将上面生成的信息输入分别到指定的输入框中，完成破解。
注意：keygen.exe生成的信息是破解  SecureCRT 的，fx.exe生成的信息是破解  SecureFX 的。

# 三、设置编码 #
- SecureCRT编码设置：
打开 SecureCRT软件，Options --> Session Options :

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/04.png)

&nbsp;&nbsp; 如上，设置为UTF-8

- SecureFX编码设置：
打开 SecureFX软件，Options --> Session Options :

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/05.png)

还需要修改配置文件：
Options -> Global Options ->  General -> Configuraition Paths :

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/06.png)


![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/07.png)

找到配置文件所在目录的Sesseion目录

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/08.png)

修改Default.ini配置内容
找到 Use UTF8"=00000000  ，将00000000  改为  00000001

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/secureCRT/09.png)

接下来新创建的连接就是以UTF8编码的了，如果之前已创建的连接，还是有乱码问题，找到此目录下指定的ip地址的 ini文件，如 192.168.1.244.ini 文件，查看是否 Use UTF8 行值为 00000000，改为 00000001




