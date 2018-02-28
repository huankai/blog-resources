---
title: Mac MySql安装
date: {{ date }}
author: huangkai
tags: 
	- Mac
	- MySql
---

## 安装 ##
从官方 [下载](http://dev.mysql.com/downloads/mysql/) mysql for mac
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下载后双击.dmg文件 , 先安装 mysql-5.5.23-osx10.6-x86.pkg （mysql主安装包）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;再安装 MySQL.prefPane （将mysql安装在系统偏好设置中） 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;再安装 MySQLStartupItem.pkg （mysql自动启动包）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上三个都安装完后，mysql就安装完成。

mysql 环境变量配置：
```text
打开终端，输入 cd ~  ,再输入open -e .base_profile 或者 nano .base_profile 
在这个文件中加入 export PATH=${path}:/usr/local/mysql/bin
然后 control + c （保存）—> Y（确认保存） —> Enter （保存并退出） 
```
###### 修改mysql默认密码：######
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安装完成后的默认root密码为空，这样很不安全，需要修改mysql密码
方式一：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开终端，将mysql的密码设置为root ,在终端输入 mysqladmin -u root -p root ，会提示输入密码，由于默认密码为空，直接回车即可完成修改，到此mysql 安装完成。

方式二：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;先用root登陆 mysql -u root -p ， 直接回车即可，执行以下命令：
```text
use mysql;
UPDATE user SET Password = PASSWORD(‘新密码') WHERE user = 'root';
FLUSH PRIVILEGES;
```
## 修改mysql 编码 ##
mysql安装好后，需要保存数据，     默认mysql的编码为latin,在保存中文时就会有乱码问题，所以需要修改编码设置。
先关闭Mysql服务 ，再执行以下命令：
```text
进入mysql目录：	
  cd /usr/local/mysql
新建文件夹etc ：   
  sudo mkdir etc
复制文件my-default.cnf 到指目录并修改文件名为my.cnf： 
  sudo cp /usr/local/mysql/support-files/my-default.cnf /usr/local/mysql/etc/my.cnf
在my.cnf 中的[mysqld]部分加入
  character-set-server=utf8
在最后加入
  [client]
  port=3306
  default-character-set=utf8
```
注意：在mysql 5.7.18版本中，在 support-files 目录中没有 my-default.cnf文件，只需要创建 etc目录后，新建 my.cnf文件，写入如下即可
```
进入Mysql根目录
  cd /usr/local/mysql
(如果该目录下没有etc文件夹，则创建)
  mkdir etc 	#创建目录
  cd etc	#进入目录
创建文件：
  vim my.cnf
写入如下内容：
  [client]
  default-character-set=utf8
  [mysqld]
  port=3306
  character_set_server=utf8
  basedir=/usr/local/mysql 			#mysql根目录
  datadir=/usr/local/mysql/data		#mysql data目录

```

## mysql 删除 ##
mac 下的mysql 的dmg只有安装文件 ，没有卸载文件 ，只能手动输入命令来删除
在删除操作之前，要停止Mysql的有关进程 ，打开终端，依次输入如下命令
```
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
vim /etc/hostconfig  (and removed the line MYSQLCOM=-YES-)
rm -rf ~/Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /var/db/receipts/com.mysql.*
```
到此，mysql删除完成
