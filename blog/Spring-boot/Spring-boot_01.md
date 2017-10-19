---
title: Spring Boot 快速入门 - 1 分钟搭建 Web 应用
date: {{ date }}
author: huangkai
tags:
categories:
    - Spring-Boot
---
Spring Boot 不是一个新的框架，它是提供一种使我们更易于创建基于 Spring 的最小或零配置的独立应用和服务的方式。

Spring 对于 Java 开发者来说一定都并不陌生，它作为目前非常流行的一个 Java 应用开发的基础框架，应用非常广泛。然而，由于其配置繁杂，各样格式的XML配置文件，着实让人头疼。

Spring Boot 的出现，可以让我们只需要非常简单的几步就可以搭建起一个基于 Spring 框架的 Web 应用程序。Spring Boot的主要目标：

- 为所有的Spring开发提供一个更快，更广泛的入门体验;
- 开箱即用，以最小或零配置的方式，使我们更专注于解决应用程序的功能需求;
- 提供一些非功能性的常见的大型项目类特性（如内嵌服务器、安全、度量、健康检查、外部化配置）;
- 绝对没有代码生成，也不需要XML配置，可以完全避免XML配置;
- 为了避免定义更多的注释配置（它将一些现有的 Spring 框架注释组合成一个简单的单个注释）;
- 提供一些默认值，以便在任何时间内快速启动新项目。


## 1. 项目依赖 ##

```java
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.hk</groupId>
    <artifactId>spring-boot-quick-start</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-boot-quick-start</name>
    <url>http://maven.apache.org</url>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.7.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```


## 2. 控制器 ##
```java
@RestController
public class HelloWorldController {
    
    @GetMapping("/")
    public String sayHello() {
        return "Hello, Spring Boot!";
    }
}
```

## 3. 主应用程序类 ##
Spring Boot 建议我们将主应用程序类置于其他类之上的根包名之下。这样就相当于隐式的的定义了注解扫描的基础搜索包名，而不需要指定 scanBasePackages 属性。
```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
    
}
```
## 4. 运行 ##
直接运行 Application 中的 main 方法：
```java
 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.7.RELEASE)
2017-10-12 13:02:06.791  INFO 7188 --- [           main] com.hk.Application                : Starting Application on FANLYCHIE-PC with PID 7188 (F:\dev\workspace\idea\spring-boot-hello-world\target\classes started by fanlychie in F:\dev\workspace\idea\spring-boot-hello-world)
2017-10-12 13:02:06.793  INFO 7188 --- [           main] com.hk.Application                : No active profile set, falling back to default profiles: default
2017-10-12 13:02:06.844  INFO 7188 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@50de0926: startup date [Sun Apr 02 13:02:06 CST 2017]; root of context hierarchy
2017-10-12 13:02:08.030  INFO 7188 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat initialized with port(s): 8080 (http)
2017-10-12 13:02:08.041  INFO 7188 --- [           main] o.apache.catalina.core.StandardService   : Starting service Tomcat
2017-10-12 13:02:08.042  INFO 7188 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.11
2017-10-12 13:02:08.121  INFO 7188 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2017-10-12 13:02:08.122  INFO 7188 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1284 ms
2017-10-12 13:02:08.254  INFO 7188 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Mapping servlet: 'dispatcherServlet' to [/]
2017-10-12 13:02:08.259  INFO 7188 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2017-10-12 13:02:08.259  INFO 7188 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2017-10-12 13:02:08.259  INFO 7188 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2017-10-12 13:02:08.259  INFO 7188 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2017-10-12 13:02:08.502  INFO 7188 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@50de0926: startup date [Sun Apr 02 13:02:06 CST 2017]; root of context hierarchy
2017-10-12 13:02:08.597  INFO 7188 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/],methods=[GET]}" onto public java.lang.String com.hk.controller.HelloWorldController.sayHello()
2017-10-12 13:02:08.599  INFO 7188 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2017-10-12 13:02:08.599  INFO 7188 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2017-10-12 13:02:08.624  INFO 7188 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-10-12 13:02:08.624  INFO 7188 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-10-12 13:02:08.661  INFO 7188 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2017-10-12 13:02:08.826  INFO 7188 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2017-10-12 13:02:08.866  INFO 7188 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-10-12 13:02:08.869  INFO 7188 --- [           main] com.hk.Application                : Started Application in 2.338 seconds (JVM running for 2.673)
```



