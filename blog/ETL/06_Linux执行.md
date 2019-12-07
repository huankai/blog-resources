---
title: ETL 利器Kettle实战应用解析系列 ————【06_Linux 执行】
date: {{ date }}
author: huangkai
tags:
	- ETL
---

生成job 并开启日志
===========

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/ETl/06_01.png)

将下载的 kittle目录文件全部上传到　linux指定文件夹上去,并将 ktr 与 kjb都上传到　data-sycn/ 目录下

```
[yind@localhost ~]$ cd data-integration/
[yind@localhost data-integration]$ ll
total 11552
-rw-r--r--.  1 yind yind     1485 Nov 14  2018 Carte.bat
-rw-r--r--.  1 yind yind     1470 Nov 14  2018 carte.sh
drwxr-xr-x.  2 yind yind      126 Dec  7 13:55 classes
drwxr-xr-x.  3 yind yind       35 Dec  7 13:55 Data Integration.app
drwxr-xr-x.  2 yind yind       59 Dec  7 13:55 Data Service JDBC Driver
drwxr-xr-x.  3 yind yind       39 Dec  7 13:55 docs
-rw-r--r--.  1 yind yind     1072 Nov 14  2018 Encr.bat
-rw-r--r--.  1 yind yind     1031 Nov 14  2018 encr.sh
-rw-r--r--.  1 yind yind     1065 Nov 14  2018 Import.bat
-rw-r--r--.  1 yind yind     2354 Nov 14  2018 import-rules.xml
-rw-r--r--.  1 yind yind     1166 Nov 14  2018 import.sh
-rw-r--r--.  1 yind yind     1118 Nov 14  2018 Kitchen.bat
-rwxr--r--.  1 yind yind     1245 Nov 14  2018 kitchen.sh
drwxr-xr-x.  2 yind yind       75 Dec  7 13:55 launcher
drwxr-xr-x.  2 yind yind    12288 Dec  7 13:55 lib
drwxr-xr-x.  6 yind yind       58 Dec  7 13:55 libswt
-rw-r--r--.  1 yind yind    13366 Nov 14  2018 LICENSE.txt
drwxr-xr-x.  2 yind yind       38 Dec  7 13:55 logs
-rw-r--r--.  1 yind yind     1106 Nov 14  2018 Pan.bat
-rwxr--r--.  1 yind yind     1211 Nov 14  2018 pan.sh
-rw-r--r--.  1 yind yind 11280251 Nov 14  2018 PentahoDataIntegration_OSS_Licenses.html
drwxr-xr-x. 30 yind yind     4096 Dec  7 13:57 plugins
-rw-r--r--.  1 yind yind     1147 Nov 14  2018 purge-utility.bat
-rw-r--r--.  1 yind yind     1241 Nov 14  2018 purge-utility.sh
drwxr-xr-x.  2 yind yind      176 Dec  7 13:57 pwd
-rw-r--r--.  1 yind yind     2741 Nov 14  2018 README-spark-app-builder.txt
-rw-r--r--.  1 yind yind     1317 Nov 14  2018 README.txt
-rw-r--r--.  1 yind yind     1456 Nov 14  2018 runSamples.bat
-rw-r--r--.  1 yind yind     1196 Nov 14  2018 runSamples.sh
drwxr-xr-x.  5 yind yind       51 Dec  7 13:57 samples
-rw-r--r--.  1 yind yind     5031 Nov 14  2018 set-pentaho-env.bat
-rw-r--r--.  1 yind yind     4602 Nov 14  2018 set-pentaho-env.sh
drwxr-xr-x.  2 yind yind       29 Dec  7 13:57 simple-jndi
-rw-r--r--.  1 yind yind     1200 Nov 14  2018 Spark-app-builder.bat
-rw-r--r--.  1 yind yind     1202 Nov 14  2018 spark-app-builder.sh
-rw-r--r--.  1 yind yind     4760 Nov 14  2018 Spoon.bat
-rw-r--r--.  1 yind yind     1111 Nov 14  2018 spoon.command
-rw-r--r--.  1 yind yind     1032 Nov 14  2018 SpoonConsole.bat
-rw-r--r--.  1 yind yind     2205 Nov 14  2018 SpoonDebug.bat
-rw-r--r--.  1 yind yind     1942 Nov 14  2018 SpoonDebug.sh
-rw-r--r--.  1 yind yind   370070 Nov 14  2018 spoon.ico
-rw-r--r--.  1 yind yind      745 Nov 14  2018 spoon.png
-rwxr--r--.  1 yind yind     7295 Nov 14  2018 spoon.sh
drwxr-xr-x.  5 yind yind       47 Dec  7 13:57 system
drwxr-xr-x.  3 yind yind     4096 Dec  7 13:57 ui
-rw-r--r--.  1 yind yind     1642 Nov 14  2018 yarn.sh
[yind@localhost data-integration]$ cd ../data-sync/
[yind@localhost data-sync]$ ll
total 72
-rw-r--r--. 1 yind yind 10155 Dec  7 15:21 sync.kjb
-rw-r--r--. 1 yind yind 37356 Dec  7 11:08 sync.ktr
-rwxr--r--. 1 yind yind   153 Dec  7 15:22 sync.sh
[yind@localhost data-sync]$ 
```

编写启动脚本:
```
#!/bin/sh

kettle_home=/home/yind/data-integration

data_sync_home=/home/yind/data-sync

# kitchen.sh 为job 启动脚本 ,pan.sh 为 转换启动脚本
$kettle_home/kitchen.sh -file=$data_sync_home/sync.kjb  -norep
```

授可执行权限:
```
[yind@localhost ~]$ cd data-integration/
[yind@localhost ~]$ chmow 744 ./kitchen.sh
[yind@localhost ~]$ chmow 744 ./pan.sh
[yind@localhost ~]$ chmow 744 ./spoon.sh

```

启动脚本:

```
[yind@localhost data-sync]$ nohup ./sync.sh > /dev/null 2>&1 &
[1] 66526
[yind@localhost data-sync]$ 
[yind@localhost data-sync]$ jps
66578 launcher.jar
66613 Jps
[yind@localhost data-sync]$  
```

关闭任务:
 ```
[yind@localhost data-sync]$ kill -9 66578
[yind@localhost data-sync]$  
```

配置 Jvm 参数:
请在 spoon.sh 文件中配置