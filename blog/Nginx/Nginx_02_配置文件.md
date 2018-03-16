---
title: Nginx 配置文件
date: {{ date }}
author: huangkai
tags:
    - Nginx
    - Linux
---

Nginx 配置文件：

```
# 定义Nginx运行的用户和用户组
user nginx nginx ;

# 启动进程,通常设置成和cpu的数量相等，cpu信息可以使用 cat /proc/cpuinfo 查看
worker_processes 8;

# 为每个进程分配cpu，上例中将8个进程分配到8个cpu，当然可以写多个，或者将一个进程分配到多个cpu
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

# 这个指令是指当一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx进程数相除，但是nginx分配请求并不是那么均匀所以最好与ulimit -n的值保持一致
worker_rlimit_nofile 102400;

#错误日志定义等级[debug/info/notice/warn/error/crit]
error_log  /var/log/nginx/error.log warn;

#pid文件路径
pid /var/run/nginx.pid;

#一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除，但是nginx分配请求并不均匀.所以建议与ulimit -n的值保持一致
worker_rlimit_nofile 65535;
	
#工作模式及连接数上限
events {
	#poll是多路复用IO(I/O Multiplexing)中的一种方式,但是仅用于linux2.6以上内核,可以大大提高nginx的性能
	use	epoll;
	
	#单个后台worker process进程的最大并发链接数 （最大连接数=连接数*进程数）
	worker_connections  102400;
	
	#尽可能多的接受请求
	multi_accept  on;
}

#设定http服务器，利用它的反向代	理功能提供负载均衡支持
http {
	#设定mime类型,类型由mime.type文件定义
	include	mime.types;
	
	default_type  application/octet-stream;
	
	#设定日志格式
	access_log	/usr/local/nginx/log/nginx/access.log;
	
	#sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用必须设为 on
	#如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
	sendfile	on;
	
	#开启目录列表访问，合适下载服务器，默认关闭。
	#autoindex  on;
	
	#防止网络阻塞
	tcp_nopush on;
	
	#keepalive超时时间，客户端到服务器端的连接持续有效时间,当出现对服务器的后继请求时,keepalive-timeout功能可避免建立或重新建立连接
	keepalive_timeout  60;
	
	#提高数据的实时响应性
	tcp_nodelay   on;
	
	#开启gzip压缩
	gzip on;
	
	gzip_min_length  1k;
	
	gzip_buffers     4 16k;
	
	gzip_http_version 1.1;
	
	gzip_comp_level  4; #压缩级别大小，最大为9，值越小，压缩后比例越小，CPU处理更快
	
	gzip_types       text/plain application/x-javascript text/css application/xml;
	
	gzip_vary on;
	
	client_max_body_size 10m;      #允许客户端请求的最大单文件字节数
	
	client_body_buffer_size 128k;  #缓冲区代理缓冲用户端请求的最大字节数
	
	proxy_connect_timeout 120;      #nginx跟后端服务器连接超时时间(代理连接超时)
	
	proxy_send_timeout 120;         #后端服务器数据回传时间(代理发送超时)
	
	proxy_read_timeout 120;         #连接成功后，后端服务器响应时间(代理接收超时)
	
	proxy_buffer_size 4k;          #设置代理服务器（nginx）保存用户头信息的缓冲区大小
	
	proxy_buffers 4 32k;           #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
	
	proxy_busy_buffers_size 64k;   #高负荷下缓冲大小（proxy_buffers*2）
	
	large_client_header_buffers  4 4k;#设定请求缓冲
	
	#客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置，一般一个请求的头部大小不会超过1k
	#不过由于一般系统分页都要大于1k，所以这里设置为分页大小。分页大小可以用命令getconf PAGESIZE取得
	client_header_buffer_size 4k;
	
	#这个将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
	open_file_cache max=102400 inactive=20s;

	#将为打开文件指定缓存，默认是没有启用的，max指定缓存数量，建议和打开文件数一致，inactive是指经过多长时间文件没被请求后删除缓存。
	open_file_cache_valid 30s;

	#这个是指多长时间检查一次缓存的有效信息。
	open_file_cache_min_uses 1;

	#open_file_cache指令中的inactive参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive
	
	#包含其它配置文件，如自定义的虚拟主机
	include  vhosts.conf;
}
```
