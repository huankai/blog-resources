---
title: Kafka_03 spring_cloud_stream_kafka 使用
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

环境：
zookeeper集群：

- 192.168.64.128:2181
- 192.168.64.129:2181
- 192.168.64.130:2181

kafka集群：
- 192.168.64.128:9092
- 192.168.64.129:9092
- 192.168.64.130:9092


spring-boot 版本：2.1.0.RELEASE
spring-cloud-stream-kafka 版本：2.1.0.RC1

# 一、生产者与消费者 #

## 1.1、生产者 ##

- 使用 hk-sso-server 项目，application.yml 配置：

```
spring:
  cloud:
    stream:
      # 当有多个binder时(如kafka、rabbitmq)，默认的binder
      default-binder: kafka
      kafka:
        binder:
          # Kafka绑定（binder）将连接到的代理（broker）列表
          brokers:
            - sjq-01:9092
            - sjq-02:9092
            - sjq-03:9092

      bindings:
		# 配置生产者，这里的output 为 org.springframework.cloud.stream.messaging.Source#OUTPUT 的值
        output:
          # 指定消息发送的目的地，消息会发送到这个主题，消费者可订阅此主题消费消息
          destination: test-destination
```

- 生产者启动类：

```
/**
 * 添加 EnableBinding 注解
 *
 * @author: kevin
 * @date: 2018-07-13 11:50
 */
@SpringBootApplication
@EnableEurekaClient
@EnableBinding(Source.class)
public class SSOServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(SSOServerApplication.class, args);
    }

}
```

- 生产者消息发送：

见 `com.hk.sso.server.kafka.controller.KafkaSourceTestController` 类

```
package com.hk.sso.server.kafka.controller;

import com.hk.commons.util.ArrayUtils;
import com.hk.core.web.JsonResult;
import lombok.Data;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.time.LocalDateTime;

@RestController
@RequestMapping("/kafka/message")
public class KafkaSourceTestController {

    private final Source source;

    @Autowired
    public KafkaSourceTestController(Source source) {
        this.source = source;
    }

    /**
     * 发送消息
     *
     * @param message message
     * @return JsonResult
     */
    @RequestMapping("/list")
    public JsonResult<Void> collectionMessage(Message message) {
        message.setDate(LocalDateTime.now());
        source.output().send(MessageBuilder.withPayload(ArrayUtils.asArrayList(message)).build());
        return JsonResult.success();
    }

    @Data
    public static class Message {
        private String id;
        private String name;
        private String title;
        private LocalDateTime date;
    }
}
```
## 1.2、消费者 ##

- 使用 hk-pms-web项目，application.yml 配置：

```
spring:
  cloud:
    stream:
      default-binder: kafka
      kafka:
        binder:
          brokers:
            - sjq-01:9092
            - sjq-02:9092
            - sjq-03:9092
      bindings:
		# input为 org.springframework.cloud.stream.messaging.Sink#INPUT 的值
        input:
          # 消息接收的主题
          destination: test-destination
          # 组名，如果不指定组名，会生成一个匿名组名，在kafka中，不同组的实例都会消费生产者发送的消息
          group: group1
```
- 消费者启动类：

```
/**
 * 添加 EnableBinding 注解 
 *
 */
@SpringCloudApplication
@EnableFeignClients(basePackages = "com.hk")
@EnableBinding(value = {Sink.class})
public class PMSApplication {

    public static void main(String[] args) {
        SpringApplication.run(PMSApplication.class, args);
    }
}
```

- 消费者接收消息:

```
package com.hk.pms.kafka;

import com.hk.commons.util.JsonUtils;
import lombok.Data;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.handler.annotation.Payload;
import org.springframework.stereotype.Component;
import java.time.LocalDateTime;
import java.util.List;

/**
 * @author: huangkai
 * @date: 2018-11-03 15:21
 */
@Component
public class TestListener {
    private Logger logger = LoggerFactory.getLogger(TestListener.class);

	/**
     * 接收消息
     * @param messages
     */
    @StreamListener(Sink.INPUT)
    public void collectionMessageReceive(@Payload List<MessageVo> messages) {
        logger.info(JsonUtils.serialize(messages, true));
    }

    @Data
    private static class MessageVo {
        private String id;
        private String name;
        private String title;
        private LocalDateTime date;
    }
}

```

## 1.3、分别启动生产者、消费者 ##

可以在 kafka-manager看到有创建的 topic（test-destination）

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_09.png)

## 1.4、发送消息 ##

可以使用postman 发送消息：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_10.png)

如下，可以看到，消费者已收到消息

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_11.png)



# 二、同一消费组在同一主题(topic)只会有一个消费者消费消息 #

生产者同上面一样，再定义两个消费者，这两个消费者属于同一组，和上面的消费者(hk-pms-web) 不在同一个组

## 2.1、新建消费者一(hk-kafka-spring-cloud-stream-consumer01)##

- application.yml配置如下：

```
server:
  port: 7771

spring:
  application:
    name: hk-kafka-spring-cloud-stream-consumer01
  main:
    banner-mode: 'off'
    allow-bean-definition-overriding: true
  cloud:
########################################################################### spring cloud stream 配置
    stream:
      default-binder: kafka
      kafka:
        binder:
          brokers:
            - sjq-01:9092
            - sjq-02:9092
            - sjq-03:9092
      bindings:
        input:
          destination: test-destination
          group: group2

```

- 启动类与消费方法：

```
package com.hk.springcloud.stream.kafka;

import com.hk.commons.util.JsonUtils;
import lombok.Data;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.messaging.handler.annotation.Payload;

import java.time.LocalDateTime;
import java.util.List;

/**
 * @author: sjq-278
 * @date: 2018-11-08 17:07
 */
@SpringBootApplication
@EnableBinding(Sink.class)
public class Consumer01Application {

    private Logger logger = LoggerFactory.getLogger(Consumer01Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Consumer01Application.class, args);
    }

    /**
     * 消息消费
     *
     * @param messages
     */
    @StreamListener(Sink.INPUT)
    public void collectionMessageReceive(@Payload List<MessageVo> messages) {
        logger.info(JsonUtils.serialize(messages, true));
    }

    @Data
    private static class MessageVo {

        private String id;

        private String name;

        private String title;

        private LocalDateTime date;
    }
}

```

## 2.2、新建消费者二(hk-kafka-spring-cloud-stream-consumer02)##

和上面一样，只有配置文件中的 端口号与 spring.application.name 改一下，<font color='red'>特别注意，spring.cloud.stream.binding.input.group 参数值一定要一样，不然不会在同一个组，发送的消息就都能接收到了，这也是这个案例的关键配置。</font> 这里不贴出来了。

## 2.3、发送消息 ##

先启动 hk-kafka-spring-cloud-stream-consumer01 和 hk-kafka-spring-cloud-stream-consumer02两个消费者，依照上面使用postman 发送两次消息，查看控制台消费消息的情况，如下图所示：

- hk-pms-web

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_12.png)

- hk-kafka-spring-cloud-stream-consumer01

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_13.png)

- hk-kafka-spring-cloud-stream-consumer02

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_14.png)

可以看到 pms 消息都所属的组(group1) 可以收到消息，hk-kafka-spring-cloud-stream-consumer01 和 hk-kafka-spring-cloud-stream-consumer02 两个消费者属于同一个组，只会有一个消费者收到消息。



