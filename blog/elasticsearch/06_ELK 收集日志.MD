---
title: ELK 收集日志
date: {{ date }}
author: huangkai
tags:
    - ElasticSearch
---

# 前言#
以下所有的收集日志都需要先安装好 Elasticsearch 与 kibana ，安装方式详见 ElasticSearch 安装 与 Kibana 安装

# 一、收集nginx日志（普通格式） #
收集 nginx 原始日志

## 1.1、安装 filebeat ##

```
filebeat:
    image: docker.elastic.co/beats/filebeat:6.8.4
    container_name: filebeat
    volumes:
      - "/etc/localtime:/etc/localtime:ro"
      - "/etc/timezone:/etc/timezone:ro"
      - "/www/docker/nginx/logs:/var/logs/nginx/:rw"  ## 这里配置的本地 nginx 日志的目录映射到 filebeat容器目录中
      - "/www/docker/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro"
      - "/var/lib/docker/containers:/var/lib/docker/containers"
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
```

## 1.2、创建 filebeat 目录，并配置 filebeat.yml  ##
```
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ mkdir /www/docker/filebeat
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ cd /www/docker/filebeat
[huangkai@iZwz93os69edbdeohzoxjeZ filebeat]$ vim filebeat.yml
```
filebeat.yml 内容如下
```
# nginx 日志输入目录
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/logs/nginx/*.log
setup.kibana:
  host: "kibana:5601"

# elasticsearch 输入，注意 elasticsearch 需要指定 elasticsearch 的服务器名称
output.elasticsearch:
   hosts: ["elasticsearch:9200"]
```

## 1.3、启动 filebeat ##
注意：在启动 filebeat 之前，需要将 `filebeat.yml` 配置文件的权限改为 644，否则，在启动的时候会报错 `Exiting: error loading config file: config file ("filebeat.yml") can only be writable by the owner but the permissions are "-rw-rw-r--" (to fix the permissions use: 'chmod go-w /usr/share/filebeat/filebeat.yml')` ，

```
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ chmod 644 filebeat.yml
```

启动 filebeat ，并查看日志，检查是否启动成功:
```
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ docker-compose up -d filebeat
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ docker-compose logs -f --tail 10 filebeat
```

## 1.4、测试 ## 
访问 nginx ，查看 nginx 的 access.log 与 error.log 日志文件
再查看 kibana 中是否收集到了 filebeat 收集的日志。

如果 出现在 filebeat-${filebeat.version}-${yyyy.mm.dd} 的 索引，且Kibana 中 Discover 菜单中的收集的数据存在，则日志收集成功。
# 二、收集 nginx 日志(JSON 格式) #

## 2.1、收集日志  ##
从 kibana 中的数据格式如下，可知,filebeat 默认是将 nginx 中的日志存放在了 message 字段中，这个字段的类型为 string ，在对于一些业务处理的时候不太方便，能否将这个消息转换为 json 字段呢？
```
"message": "120.230.125.60 - - [29/Feb/2020:20:23:03 +0800] \"GET /favicon.ico HTTP/1.1\" 200 4286 \"http://119.23.31.32/login\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36\" \"-\"",
```

通过查看nginx 的配置文件(nginx.conf)中可知，nginx默认的日志输出格式配置`$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" "$http_x_forwarded_for"`，那么，我们可以修改nginx 的输出格式如下(重新定义一个 log_format 名称为 json ，并指定access_log 的输出名称为 json)：

```
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  json  '{"remote_addr":"$remote_addr","remote_user": "$remote_user","time_local": "[$time_local]","request_method": "$request_method", '
                      '"request_uri":"$request_uri","status": $status,"body_sent": $body_bytes_sent,"http_referer":"$http_referer", '
                      '"http_user_agent":"$http_user_agent","http_forwarded_for": "$http_x_forwarded_for"}';
    access_log  /var/log/nginx/access.log json;


    ... ## nginx 其它配置
```

配置 filebeat.yml ，内容如下:
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/logs/nginx/*.log
  json.keys_under_root: true  ## keys_under_root可以让字段位于根节点，默认为false
  json.add_error_key: true  ## 将解析错误的消息记录储存在error.message字段中

setup.kibana:
  host: "kibana:5601"
output.elasticsearch:
   hosts: ["elasticsearch:9200"]
```

清除 nginx 日志，清除 Kibana 的索引数据，重启 nginx,重启 filebeat ，访问nginx ，可以在 kibana 中查看数据已收集。

    见图(2)。


## 2.2、修改索引名称  ##

filebeat 收集的日志存储到 elasticsearch 后，对应 的 Elasticsearch 中的索引名为 filebeat-${filebeat.version}-${yyyy.MM.dd} ，如何修改自定的索引名呢？

如下，指定 output.elasticsearch.index、setup.template.name、setup.template.pattern 即可。

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/logs/nginx/*.log
  json.keys_under_root: true
  json.add_error_key: true
setup.kibana:
  host: "kibana:5601"

output.elasticsearch:
   hosts: ["elasticsearch:9200"]
   index: "nginx-%{[beat.version]}-%{+yyyy.MM}"

setup.template.name: "nginx"
setup.template.pattern: 'nginx-*'  ## 匹配 nginx- 开头的索引使用自定的的索引名
setup.template.enabled: false ## 关闭filebeat内置模板
```

重启 filebeat ,访问 nginx,如果在kibana 中出现了 `nginx-%{[beat.version]}-%{+yyyy.MM}` 格式的索引，索引中存在收集nginx的数据，自定义索引成功，如图3。

# 三、收集 nginx 日志(正确日志与错误日志) #
访问 nginx 时，会将正确日志写入到 access.log 中，错误日志写入到 error.log 中
需求:将 access.log 中的内容收集到 nginx-access-${yyyy-MM} 索引中，将 error.log 中的内容收集到 nginx-error-${yyyy-MM} 索引中。

步骤：
1、使用压力测试工具 ab 模拟请求 `yum -y install httpd-tools`
2、清除 nginx access.log 与 error.log内容
3、配置 filebeat.yml
```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/logs/nginx/access.log
  json.keys_under_root: true
  json.add_error_key: true
  tags: ['access']  ## 定义 access 标签
- type: log
  enabled: true
  paths:
    - /var/logs/nginx/error.log
  tags: ['error']
setup.kibana:
  host: "kibana:5601"

output.elasticsearch:
   hosts: ["elasticsearch:9200"]
   indices:
    - index: "nginx-access-%{+yyyy-MM}"
      when.contains:
        tags: "access"  ## 当 tags 中包含 access 时，收集到 此 索引中
    - index: "nginx-error-%{+yyyy-MM}"
      when.contains:
        tags: "error"  ## 当 tags 中包含 error 时，收集到 此 索引中
setup.template.name: "nginx"
setup.template.pattern: 'nginx-*'
```

重启 filebeat ，使用 ab 压力测试工具测试 
```
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ ab -n 10 -c 10 https://www.kevin.pet/emi/xxxx
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking www.kevin.pet (be patient).....done


Server Software:        nginx
Server Hostname:        www.kevin.pet
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128

Document Path:          /emi/xxxx
Document Length:        150 bytes

Concurrency Level:      10
Time taken for tests:   0.018 seconds
Complete requests:      10
Failed requests:        0
Write errors:           0
Non-2xx responses:      10
Total transferred:      2950 bytes
HTML transferred:       1500 bytes
Requests per second:    542.92 [#/sec] (mean)
Time per request:       18.419 [ms] (mean)
Time per request:       1.842 [ms] (mean, across all concurrent requests)
Transfer rate:          156.41 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        3   11   2.7     12      12
Processing:     1    1   0.6      1       2
Waiting:        0    1   0.6      1       2
Total:          4   12   3.0     13      14

Percentage of the requests served within a certain time (ms)
  50%     13
  66%     14
  75%     14
  80%     14
  90%     14
  95%     14
  98%     14
  99%     14
 100%     14 (longest request)
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$
```

查看 kibana 中是否存在 `nginx-access-${yyyy-MM}` 和 `nginx-access-${yyyy-MM}` 两个索引，索引中是否存在数据。
## 四、收集 java 日志 ##

java 日志中，如果未抛出异常信息时，日志是每一行记录的， 但抛出异常的日志就不能确定有多少行了，所以，需要配置能自动识别将异常信息也作为一行记录。

假设 inputs 的日志文件内容为 /var/logs/elasticsearch/elastic.log ,该日志的内容就是Java所写的。配置内容如下

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/logs/elasticsearch/elastic.log
  multiline.pattern: '^\['
  multiline.negate: true
  multiline.match: after
setup.kibana:
  host: "kibana:5601"
output.elasticsearch:
   hosts: ["elasticsearch:9200"]
   indices:
    - index: "elastic-%{+yyyy-MM}"
setup.template.name: "elastic"
setup.template.pattern: 'elastic-*'
```

以上主要是开启了三个参数  `multiline.pattern` 、`multiline.negate`、 `multiline.match` 这三个参数的配置可参考文档: https://www.elastic.co/guide/en/beats/filebeat/6.8/multiline-examples.html
## 五、收集 docker 容器日志 ##

1、安装 docker，并启动一个docker 容器应用,查看容器id 如下
```
[huangkai@iZwz93os69edbdeohzoxjeZ hk]$ docker inspect hk-oauth2-server |grep Id
        "Id": "78c90593ebc14222af23effd6519048e432fab7ed856cf698408557451aefcfe",
[huangkai@iZwz93os69edbdeohzoxjeZ hk]$ 
```

2、配置 filebeat.yml 内容如下：
```
filebeat.inputs:
- type: docker
  containers.ids:
      - "78c90593ebc14222af23effd6519048e432fab7ed856cf698408557451aefcfe"
multiline.pattern: ^\[
multiline.negate: true
multiline.match: after
setup.kibana:
  host: "172.17.0.1:5601"

output.elasticsearch:
   hosts: ["172.17.0.1:9200"]
   index: "docker-%{+yyyy-MM}"
setup.template.name: "docker"
setup.template.pattern: "docker-*"
```
配置参数参考 https://www.elastic.co/guide/en/beats/filebeat/6.8/filebeat-input-docker.html

3、重启 filebeat,访问 docker 容器应用，使用 kibana 查看 是否有 docker-${yyyy-MM} 的索引创建成功，并有内容添加到容器中.

如图4
## 六、 filebeat 模块收集日志 ##
上面的例子中都是自定义收集日志的，其实，filebeat 中存在内置的收集日志模块,进入filebeat目录可以看到存在  `modules.d` 文件夹，该文件夹就是内置的模块.默认情况下，该目录下的所有模块都是未启动状态，文件名都是 .disabled 结尾的。

```
[huangkai@iZwz93os69edbdeohzoxjeZ filebeat]$ ll
total 36776
drwxr-x---  2 root     root         4096 Mar  1 10:26 data
-rw-r--r--  1 huangkai huangkai   146747 Oct 16 13:14 fields.yml
-rwxr-xr-x  1 huangkai huangkai 37216593 Oct 16 13:17 filebeat
-rw-r--r--  1 huangkai huangkai    70172 Oct 16 13:14 filebeat.reference.yml
-rw-r--r--  1 root     root          383 Mar  1 10:20 filebeat.yml
drwxr-xr-x  4 huangkai huangkai     4096 Oct 16 13:12 kibana
-rw-r--r--  1 huangkai huangkai    13675 Oct 16 13:10 LICENSE.txt
drwx------  2 root     root         4096 Mar  1 10:21 logs
drwxr-xr-x 21 huangkai huangkai     4096 Oct 16 13:14 module
drwxr-xr-x  2 huangkai huangkai     4096 Oct 16 13:14 modules.d
-rw-r--r--  1 huangkai huangkai   163444 Oct 16 13:10 NOTICE.txt
-rw-r--r--  1 huangkai huangkai      802 Oct 16 13:17 README.md
[huangkai@iZwz93os69edbdeohzoxjeZ filebeat]$ ll modules.d/
total 76
-rw-r--r-- 1 huangkai huangkai  371 Oct 16 13:14 apache2.yml.disabled
-rw-r--r-- 1 huangkai huangkai  175 Oct 16 13:14 auditd.yml.disabled
-rw-r--r-- 1 huangkai huangkai 1250 Oct 16 13:14 elasticsearch.yml.disabled
-rw-r--r-- 1 huangkai huangkai  269 Oct 16 13:14 haproxy.yml.disabled
-rw-r--r-- 1 huangkai huangkai  546 Oct 16 13:14 icinga.yml.disabled
-rw-r--r-- 1 huangkai huangkai  371 Oct 16 13:14 iis.yml.disabled
-rw-r--r-- 1 huangkai huangkai  257 Oct 16 13:14 iptables.yml.disabled
-rw-r--r-- 1 huangkai huangkai  396 Oct 16 13:14 kafka.yml.disabled
-rw-r--r-- 1 huangkai huangkai  188 Oct 16 13:14 kibana.yml.disabled
-rw-r--r-- 1 huangkai huangkai  563 Oct 16 13:14 logstash.yml.disabled
-rw-r--r-- 1 huangkai huangkai  189 Oct 16 13:14 mongodb.yml.disabled
-rw-r--r-- 1 huangkai huangkai  368 Oct 16 13:14 mysql.yml.disabled
-rw-r--r-- 1 huangkai huangkai  569 Oct 16 13:14 nginx.yml.disabled
-rw-r--r-- 1 huangkai huangkai  388 Oct 16 13:14 osquery.yml.disabled
-rw-r--r-- 1 huangkai huangkai  192 Oct 16 13:14 postgresql.yml.disabled
-rw-r--r-- 1 huangkai huangkai  463 Oct 16 13:14 redis.yml.disabled
-rw-r--r-- 1 huangkai huangkai  190 Oct 16 13:14 suricata.yml.disabled
-rw-r--r-- 1 huangkai huangkai  574 Oct 16 13:14 system.yml.disabled
-rw-r--r-- 1 huangkai huangkai  195 Oct 16 13:14 traefik.yml.disabled
[huangkai@iZwz93os69edbdeohzoxjeZ filebeat]$ 
```

要启动 filebeat 的内置 module ，步骤如下(以下以 nginx 为例)：
官网地址 : https://www.elastic.co/guide/en/beats/filebeat/6.8/filebeat-module-nginx.html

1、如果 ElasticSearch 版本小于 6.7，则需要安装 [ingest-user-agent](https://www.elastic.co/guide/en/elasticsearch/plugins/6.8/ingest-user-agent.html) 和 [ingest-geoip](https://www.elastic.co/guide/en/elasticsearch/plugins/6.8/ingest-geoip.html) 插件，这的 elasticsearch版本为 6.8.4，可以不用安装这两个插件，安装方式这里略。

2、配置`filebeat.yml`
```
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
  #reload.period: 10s
setup.kibana:
  host: "172.17.0.1:5601"

output.elasticsearch:
   hosts: ["172.17.0.1:9200"]
   indices:
    - index: "nginx-access-%{+yyyy-MM}"
      when.contains:
        fileset.name: "access"   ## 内置nginx模块可以用 fileset.name 区分是否为access日志
    - index: "nginx-error-%{+yyyy-MM}"
      when.contains:
        fileset.name: "error" 
setup.template.name: "nginx"
setup.template.pattern: "nginx-*"
```

3、查看指定的模块功能
第二步骤配置好后，重启 filebeat,进入 filebeat docker 容器中，
查看 开启的模块功能，如下所示，现在暂无开启的模块.

```
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ docker-compose exec filebeat bash
bash-4.2$ filebeat modules list
Enabled:

Disabled:
apache2
auditd
elasticsearch
haproxy
icinga
iis
iptables
kafka
kibana
logstash
mongodb
mysql
nginx
osquery
postgresql
redis
suricata
system
traefik
bash-4.2$ 
```

4、开启模块(nginx 为例)
4.1、使用命令行:
```
bash-4.2$ filebeat modules enable nginx
Enabled nginx
bash-4.2$ filebeat modules list        
Enabled:
nginx

Disabled:
apache2
auditd
elasticsearch
haproxy
icinga
iis
iptables
kafka
kibana
logstash
mongodb
mysql
osquery
postgresql
redis
suricata
system
traefik
bash-4.2$ 
```

4.2、也可以直接重命名需要启动的模块,如:
```
bash-4.2$ cd modules.d/
bash-4.2$ ls -la
total 92
drwxrwx--- 1 root filebeat 4096 Mar  1 10:57 .
drwxr-x--- 1 root filebeat 4096 Oct 16 13:17 ..
-rw-r----- 1 root filebeat  371 Oct 16 13:17 apache2.yml.disabled
-rw-r----- 1 root filebeat  175 Oct 16 13:17 auditd.yml.disabled
-rw-r----- 1 root filebeat 1250 Oct 16 13:17 elasticsearch.yml.disabled
-rw-r----- 1 root filebeat  269 Oct 16 13:17 haproxy.yml.disabled
-rw-r----- 1 root filebeat  546 Oct 16 13:17 icinga.yml.disabled
-rw-r----- 1 root filebeat  371 Oct 16 13:17 iis.yml.disabled
-rw-r----- 1 root filebeat  257 Oct 16 13:17 iptables.yml.disabled
-rw-r----- 1 root filebeat  396 Oct 16 13:17 kafka.yml.disabled
-rw-r----- 1 root filebeat  188 Oct 16 13:17 kibana.yml.disabled
-rw-r----- 1 root filebeat  563 Oct 16 13:17 logstash.yml.disabled
-rw-r----- 1 root filebeat  189 Oct 16 13:17 mongodb.yml.disabled
-rw-r----- 1 root filebeat  368 Oct 16 13:17 mysql.yml.disabled
-rw-r----- 1 root filebeat  569 Oct 16 13:17 nginx.yml
-rw-r----- 1 root filebeat  388 Oct 16 13:17 osquery.yml.disabled
-rw-r----- 1 root filebeat  192 Oct 16 13:17 postgresql.yml.disabled
-rw-r----- 1 root filebeat  463 Oct 16 13:17 redis.yml.disabled
-rw-r----- 1 root filebeat  190 Oct 16 13:17 suricata.yml.disabled
-rw-r----- 1 root filebeat  574 Oct 16 13:17 system.yml.disabled
-rw-r----- 1 root filebeat  195 Oct 16 13:17 traefik.yml.disabled
bash-4.2$ mv mysql.yml.disabled mysql.yml   
bash-4.2$ cd ..
bash-4.2$ filebeat modules list
Enabled:
mysql
nginx

Disabled:
apache2
auditd
elasticsearch
haproxy
icinga
iis
iptables
kafka
kibana
logstash
mongodb
osquery
postgresql
redis
suricata
system
traefik
bash-4.2$ 
```
以上就开启了 mysql 的内置模块


5、禁用模块

如下，禁用 mysql 模块
```
bash-4.2$ filebeat modules disable mysql
Disabled mysql
bash-4.2$ filebeat modules list
Enabled:
nginx

Disabled:
apache2
auditd
elasticsearch
haproxy
icinga
iis
iptables
kafka
kibana
logstash
mongodb
mysql
osquery
postgresql
redis
suricata
system
traefik
bash-4.2$ 
```


6、收集内置模块日志(nginx)
nginx 的 access.log 使用内置模块可以将日志自动转换为 json 格式，方便解析，虽然上面我们也自定义了 nginx的格式转换成了 json 格式，但比较麻烦，这里使用内置的nginx 模块。

将 nginx 的日志格式改为默认的,如下，将access_log 的名称改为 main

```
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    log_format  json  '{"remote_addr":"$remote_addr","remote_user": "$remote_user","time_local": "[$time_local]","request_method": "$request_method", '
                      '"request_uri":"$request_uri","status": $status,"body_sent": $body_bytes_sent,"http_referer":"$http_referer", '
                      '"http_user_agent":"$http_user_agent","http_forwarded_for": "$http_x_forwarded_for"}';
    access_log  /var/log/nginx/access.log main;
```

使用 root 账号进入 filebeat 容器内，
```
[huangkai@iZwz93os69edbdeohzoxjeZ docker]$ docker-compose exec --user root filebeat bash
```
修改 ${filebeat_home}/moduldes.d/nginx.yml 配置文件，内容如下：
```
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/logs/nginx/access.log"]  ## 注意，这里的格式要写成数组
  error:
    enabled: true
    var.paths: ["/var/logs/nginx/error.log"]
```

重启 filebeat 和 nginx，访问 nginx ，检查 kibana 中是否存在 nginx-access-${yyyy-MM} 和 nginx-error-${yyyy-MM} 的索引，查看两个索引中是否有数据

## 七、 使用 redis 作为缓存收集日志 ##

使用 nginx + filebeat + redis + logstash + elasticsearch + kibana 收集 nginx 日志.
filebeat 收集 nginx 日志，保存到 redis 中， Logstash 从 redis 中消费日志保存到 elasticsearch 中， kibana 显示日志内容。

1、nginx 日志格式配置：

```
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log main;

    ... ### 其它 nginx 配置
```

2、接着第六节的，filebeat 开始 nginx 模块后，filebeay.yml 配置内容如下:

```
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
  #reload.period: 10s
setup.kibana:
  host: "172.17.0.1:5601"
output.redis:
  hosts: ["172.0.0.1:6350"]
  password: "redisa$min302"
  db: 0
  timeout: 5
  keys:
    - key: "access_list"   
      when.contains:
        fileset.name: "access"
    - key: "error_list"  
      when.contains:
        fileset.name: "access"
```


3、安装、启动redis 
略

4、安装，启动 logstash


## 八、 使用 Kafka 收集日志 ##

