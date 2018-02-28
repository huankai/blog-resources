---
title: Spring Boot 添加 Thymeleaf 支持
date: {{ date }}
author: huangkai
tags:
    - Spring-Boot
---
Spring Boot 对 Thymeleaf 模板引擎提供了自配置的良好支持。Spring Boot 1.5.7.RELEASE 版本默认使用的是 Thymeleaf 2.0+，本文使用 Thymeleaf 3.0+ 版本，在 pom.xml 中添加以下声明：

```
<properties>
	<thymeleaf.version>3.0.5.RELEASE</thymeleaf.version>
	<thymeleaf-layout-dialect.version>2.2.1</thymeleaf-layout-dialect.version>
</properties>
```
然后在 dependencies 添加 Thymeleaf 依赖声明:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

**控制器:**
```
@Controller
public class WelcomeController {
    
    @GetMapping("/")
    public String welcome(ModelMap model) {
        model.put("message", "Hello Thymeleaf!");
        return "index";
    }
}
```

**模板文件:**

Spring Boot 对 Thymeleaf 模板引擎提供了自动配置的支持(详见 [ThymeleafProperties](https://github.com/spring-projects/spring-boot/blob/v1.5.7.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafProperties.java#L37))。我们只需遵循约定，在/src/main/resources/templates/目录创建相应的页面模板文件（*.html）即可
 \#src/main/resources/templates/index.html

```
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>首页</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <link rel="stylesheet" type="text/css" th:href="@{/css/main.css}">
</head>
<body>
    <h1 th:text="${message}"></h1>
</body>
</html>
```
**静态文件:**
Spring Boot 默认将静态资源文件映射到类路径下的目录包括(详见  [ResourcesProperties](https://github.com/spring-projects/spring-boot/blob/v1.5.7.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java#L44))：
- /META-INF/resources/
- /resources/
- /static/
- /public/

因此我们可以将 css、js、images 等静态资源文件放在/src/main/resources/static/目录下.

\#src/main/resources/static/css/main.css
```
body {
    padding: 0;
    color: #444;
    width: 280px;
    margin: 100px auto;
    font-family: SimSun;
    background-color: #FBFBFB;
    text-shadow: rgba(50,50,50,0.3) 2px 2px 3px;
}
```

**主应用程序类:**

```
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
    
}
```

**模板文件和静态资源文件的缓存问题:**

当修改 css、js 等静态资源文件的内容或模板文件的内容时，刷新客户端浏览器，发现内容还是老的，说明 Spring Boot 内置的 Servelt 容器并没有实时重新加载修改过的文件内容。你只能在每次修改静态资源文件时，虽然不需要重启服务，但是你要重新编译一次，IntelliJ IDEA 中按一次 Ctrl + F9 即可。

有关 Thymeleaf 基本使用可查看[ Thymeleaf 教程](https://huankai.github.io/2018/02/22/Thymeleaf_01/) 