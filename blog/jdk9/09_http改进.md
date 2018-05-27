---
title: 全新的HTTP API
date: {{ date }}
author: huangkai
tags:
    - JDK9
---

HTTP(超文体传输协议)，早上1997年被采用在目前的 HTTP1.1版本中，直到 2015年，HTTP 2 才成为标准。


HTTP 1.1 和 HTTP 2 主要区别是如何在客户端和服务端之间构建和传输数据。HTTP1.1依赖于请求/响应周期，HTTP 2 允许服务器 `push`数据 ，它可以发送比客户端请求更多的数据，这使得它可以优先处理并发送首先加载页面至关重要的数据。

JDK 9 中有新的方式来处理HTTP调用 ，它提供了一个新的HttpClient 替代旧的，blocking 模式的HttpURLConnection(HTTP1.0时代，并使得了协议无关的方法)，并提供了对 HTTP2 和 Websocket的支持。此外，HTTPClient 还提供了处理HTTP2的相关特性，比如流和服务器推送等功能。

全新的HTTPClient 需要在 module-info.java 中添加： **requires jdk.incubator.httpclient;**
```
public static void main(String[] args) throws IOException, InterruptedException {
    HttpClient httpClient = HttpClient.newHttpClient();
    HttpRequest httpRequest = HttpRequest.newBuilder(URI.create("https://www.baidu.com")).GET().build();
    HttpResponse<String> response = httpClient.send(httpRequest, HttpResponse.BodyHandler.asString());
    System.out.println(response.statusCode());
    System.out.println(response.version().name());
    System.out.println(response.body());	
    }

```