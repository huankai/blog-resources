---
title: Kafka_04_JAVA_API_示例
date: {{ date }}
author: huangkai
tags:
    - Kafka
---

# 需求一、生产者生产消息，消费者消费消息 #

生产者代码：

```
package com.hk.kafka.producer.example;

import org.apache.kafka.clients.producer.*;
import org.apache.kafka.clients.producer.internals.DefaultPartitioner;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import sun.rmi.runtime.Log;

import java.util.Properties;

/**
 * @author: huangkai
 * @date: 2018-08-30 15:58
 */
public class SimpleProducer {

    private static final Logger Logger = LoggerFactory.getLogger(SimpleProducer.class);

    public static void main(String[] args) throws InterruptedException {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "sjq-01:9092,sjq-02:9092,sjq-03:9092");
        props.put(ProducerConfig.ACKS_CONFIG, "all");//等待分区的所有副本应答，才表示此消息发送成功
        props.put(ProducerConfig.RETRIES_CONFIG, 0); // 生产者从服务器收到临时性错误时，生产者重发消息的次数
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);// 一批消息处理大小
        props.put(ProducerConfig.LINGER_MS_CONFIG, 1); // 请求延时
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432); // 发送缓存区内存大小
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName()); // key序列化
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName()); // value序列化

        // 指定  partitioner 类
//        props.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, DefaultPartitioner.class);

        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        boolean running = true;
        int index = 1;
        while (running) {
            Thread.sleep(800);
            String message = "Send message " + (index++);
            Logger.info("Send message {}...", message);
// 有回调的生产者发送消息，发送消息到 test3主题
            producer.send(new ProducerRecord<>("test3", "luck", message), new Callback() {
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception != null) {
                        Logger.error(exception.getMessage());
                    } else {
                        Logger.info("Send success ,partition is {},", metadata.partition());
                    }
                }
            });

//            producer.send(new ProducerRecord<>("test3", "luck", message));// 无回调发送
        }
        Logger.info("finish");
        producer.close();
    }
}

```

消费者代码：

```
package com.hk.kafka.consumer.example;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

/**
 * @author: huangkai
 * @date: 2018-08-30 16:17
 */
public class SimpleConsumer {

    private static final Logger LOGGER = LoggerFactory.getLogger(SimpleConsumer.class);

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "sjq-01:9092,sjq-02:9092,sjq-03:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test-consumer-group");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true); // 是否自动确认offset
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 1000); // 自动确认offset的时间间隔
//        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 30000);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("test3")); //订阅 test3主题
        LOGGER.info("等待消费...");
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                LOGGER.info("offset = {},topic = {},partition = {}, key = {}, value = {}", record.offset(), record.topic(), record.partition(), record.key(), record.value());
            }
        }
    }

}

```

测试结果 ：
- 如果主题 test3 每个partition 只有一个副本数，也就是它本身，如果此 partition 所在的 broker 宕机，消息丢失

 如下图所示：启动消费者-->启动生产者 --> 生产者不断生产消息，消费者不断消费消息 --> 查看 test3 topic 只有一个 partition 一个副本 --> 模拟 broker 宕机 --> 查看生产者继续生产消息 --> 查看消费者未能消费消息  --> 模拟恢复宕机的 broker --> 查看生产者正常生产消息 --> 查看消费者是从  68 开始消费的，中间的消息丢失。

 ![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_15.gif)

- 如果主题目 test3 所在的 partition 的 Broker 宕机，消费此 partition 消费者无法消费消息(每个消费者会消费指定partition的消息)，如果副本数只有一个，消息丢失，如果有多个，需要将此宕机的 Broker启动，消费者能正常消费消息（partition的副本机制）
- 生产者的 `ProducerConfig.RETRIES_CONFIG` 参数配置不等于0 时，如果broker 宕机，消息会重试（可查看 [消息重试分析](https://blog.csdn.net/feelwing1314/article/details/81206506)），如果所宕机的 topic的partition 的副本数只有一个，且partition 就是宕机的 broker，消息丢失（消息无法到达kafka），否则，消息不会丢失（kfaka会选举其它的副本作为宕机的partition，处理消息），当宕机的broker 正常后，可以消费消息。

# 案例二：测试同一个消费者组中的消费者，同一时刻只能有一个消费者消费 #

分别修改192.168.64.129，192.168.64.130 的${KAFKA_HOME}/config/consumer.properties文件 group.id 参数为任意相同值，也可不修改，默认都为 test-consumer-group

分别在 192.168.64.129，192.168.64.130 上启动kafka 消费者：


```
[huangkai@sjq-02  kafka_2.12-2.0.0]$ kafka-console-consumer.sh --bootstrap-server sjq-01:9092 --topic test -consumer.config ./config/consumer.properties
```

在 192.168.64.128 启动 kafka生产者：
```
[huangkai@sjq-01  kafka_2.12-2.0.0]$ kafka-console-producer.sh --broker-list sjq-01:9092 --topic test
```

![](https://raw.githubusercontent.com/huankai/blog-resources/master/photos/kafka/kafka_08.png)

查看 在同一时刻，只用一个消费者收到了消息。