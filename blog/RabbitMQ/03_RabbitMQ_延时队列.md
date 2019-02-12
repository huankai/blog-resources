---
title: 03_RabbitMQ 延迟队列
date: {{ date }}
author: huangkai
tags:
    - RabbitMQ
---

# 一、延迟队列 #

Delayed Message Plugin 是rabbitmq官网提供的延迟队列插件，其原理是通过 exchange 来实现延迟功能，可以根据 message 的 `x-delay` 头设置延迟时间，时间到达后发送到queue ，而被 queue 消费。

应用场景：定时任务、定时发送消息/邮件、订单状态延时处理(如12306购票，用户在指定时间未付款，则将订单设置为失效)，等等

下载插件 : [http://www.rabbitmq.com/community-plugins.html](http://www.rabbitmq.com/community-plugins.html) 
找到 `rabbitmq_delayed_message_exchange` 下载对应的 rabbitmq版本的插件，下载完成后解压，将解压后的 `ez` 结尾的文件复制到 rabbitmq 安装目录 的 plugins 目录下，使用docker 安装的目录在 /plugins ,可使用 docker cp 命令复制，如下：

```
[huangkai@localhost docker]$ ll
total 96
-rw-------  1 huangkai huangkai  3887 Feb 11 14:30 docker-compose.yml
drwxrwxr-x  5 huangkai huangkai  4096 Feb 11 14:28 rabbitmq
-rw-r--r--  1 huangkai huangkai 43958 Jan 16  2018 rabbitmq_delayed_message_exchange-20171201-3.7.x.ez
[huangkai@localhost docker]$ docker cp ./rabbitmq_delayed_message_exchange-20171201-3.7.x.ez 52a5cd1adc2e:/plugins
[huangkai@localhost docker]$ docker-compose exec rabbitmq bash
root@52a5cd1adc2e:/#
root@52a5cd1adc2e:/# ls plugins|grep rabbitmq_delayed_message_exchange
rabbitmq_delayed_message_exchange-20171201-3.7.x.ez
root@52a5cd1adc2e:/# rabbitmq-plugins enable rabbitmq_delayed_message_exchange  # 启动插件
The following plugins have been configured:
  rabbitmq_delayed_message_exchange
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@52a5cd1adc2e...
The following plugins have been enabled:
  rabbitmq_delayed_message_exchange

started 1 plugins.
root@52a5cd1adc2e:/#
```
# 二、测试 #

基于 spring-cloud-stream-rabbit 测试
## 2.1、生产者 ##

```
################ application.yml 配置文件部分内容如下：
...
spring
  cloud:
    stream:
      default-binder: rabbit
      bindings:
        delayed_output:
          destination: delayed_stream
      rabbit:
        bindings:
          delayed_output:
            producer:
#            配置是延时队列
              delayed-exchange: true
```

```
################# 生产者自定义接口

package com.hk.example.rabbit.stream;

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;


public interface DelayedOutput {

    String OUTPUT = "delayed_output";

    @Output(OUTPUT)
    MessageChannel output();
}
```

```
################# 生产者启动类及发送消息

package com.hk.example.rabbit.stream;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.context.annotation.Bean;
import org.springframework.integration.annotation.InboundChannelAdapter;
import org.springframework.integration.annotation.Poller;
import org.springframework.integration.core.MessageSource;
import org.springframework.messaging.support.GenericMessage;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@SpringBootApplication
@EnableBinding(value = {DelayedOutput.class})
public class ProducerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class, args);
    }

    /**
     * 延时队列
     */
    @Bean
    @InboundChannelAdapter(value = DelayedOutput.OUTPUT, poller = @Poller(fixedRate = "10000", maxMessagesPerPoll = "1"))
    public MessageSource<Date> topicSend() {
        System.out.println("delayed_output sendMessage...");
        Map<String, Object> headers = new HashMap<>();
        headers.put("x-delay", 3000);
        //延时队列需要在 header中添加 x-delay指定延时的时间，单位为毫秒,这里表示3秒后消费者可以从 queue 中消费消息
        return () -> new GenericMessage<>(new Date(), headers);
    }
}

```


## 2.2、消费者 ##

```
################ application.yml 配置文件部分内容如下：
spring:
  cloud:
    stream:
      bindings:
        delayed_input:
          destination: delayed_stream
      rabbit:
        bindings:
          delayed_input:
            consumer:
              delayed-exchange: true
```

```
package com.hk.example.rabbit.stream;

import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;

public interface DelayedInput {

    String INPUT = "delayed_input";

    @Input(INPUT)
    SubscribableChannel input();

}

```

```
################# 消费者启动类及接收消息
package com.hk.example.rabbit.stream;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.messaging.handler.annotation.Payload;

import java.time.LocalDateTime;
import java.util.Date;

@SpringBootApplication
@EnableBinding(value = {DelayedInput.class})
public class Consumer01Application {

    private static final Logger LOGGER = LoggerFactory.getLogger(Consumer01Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Consumer01Application.class, args);
    }

    /**
     * 延时队列消费
     */
    @StreamListener(DelayedInput.INPUT)
    public void delayedReceive(@Payload Date date) {
        LOGGER.info("消息接收时间 : {}", LocalDateTime.now());
        LOGGER.info("Consumer01Application delayedReceive message : {}", date);
    }
}

```


启动消费者、生产者，查看控制台打印信息如下，消息接收时间与消息内容的时间相差 3 秒：

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/rabbitmq/rabbitm_01.png)

查看rabbitmq 管理控制台，delayed_stream 的类型为 x-delayed-message

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/rabbitmq/rabbitm_02.png)