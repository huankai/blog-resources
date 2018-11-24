---
title: Tomcat 配置 POST 请求大小
date: {{ date }}
author: huangkai
tags:
    - tomcat
---

今天项目开发的时候遇到一个问题，在一个巨大的表单提交时，这个表单包括一些参数、还有一个使用富文本编辑器导入的word 文档，后台使用 `request.getParameter(parameterName)` 获取不到值，使用浏览器查看表单请求参数，发现都传递了，排除客户端的问题。后来找了相关资料发现是 tomcat 的配置问题。通过配置 ${TOMCAT_HOME}/conf/server.xml 的`maxPostSize` 参数解决问题，默认值为 2M 。注意不能设置为0，设置为0 所有的参数都不能接收到值，应该配置为一个小于0的数来禁用POST参数的大小，配置如下：

```
...
<Connector port="8080" protocol="HTTP/1.1"
   connectionTimeout="20000" maxPostSize="-1" URIEncoding="utf-8"
   redirectPort="8443" />
...
```

参考资料： http://tomcat.apache.org/tomcat-8.0-doc/config/http.html