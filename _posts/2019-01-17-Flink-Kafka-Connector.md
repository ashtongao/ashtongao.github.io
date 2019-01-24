---
layout:     post
title:      Flink-Kafka-Connector
subtitle:   Flink-Kafka-Connector
date:       2019-01-08
author:     Ashton
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - Flink
    - Java
    - kafka
---


## 基本介绍

Kafka 是常见的消息队列，在Flink中，提供了特殊的Kafka Connector来从Kafka topic中读写数据。

## Kafka Consumer

Flink的Kafka consumer叫做FlinkKafkaConsumer08，后面的08代表仅支持09或者09以上的Kafka版本。

使用

```java
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");

// Kafka 0.8 版本才需要
properties.setProperty("zookeeper.connect", "localhost:2182");
properties.setProperty("group.id", "test");

DataStream<String> strem = env.addSource(new FlinkKafkaConsumer08<>("topic", new SimpleStringSchema(), properties));
```

constructor的参数

1. topic 名称
2. DeserializationSchema或者KeyedDeserializationSchema用于反序列化Kafka数据
3. Kafka Comsumer配置
    * bootstrap.ervers
    * zookeeper.connect
    * group.id，是consumer group id


## DeserializationSchema

DeserializationSchema 可以让 Flink Kafka Consumer 知道如何将二进制数据转化为Java对象，每一个Kafka消息都会使用反序列化方法`T deserialize(byte[] message)`来解析，从而获得从Kafka中传递过来的值。

`AbstractDeserializationSchema.java` 描述了如何将Java/Scala类型转化为Flink类型，如果需要实现 `vanilla DeserializationSchema`，则需要实现 `getProducedType(...)` 方法

Flink提供了以下的schemas

1. `TypeInformationSerializationSchema` ，基于 Flink's TypeInformation，可以很方便读写
2. `JsonDeserializationSchema`，将JSON序列化为ObjectNode对象，可以通过ObjectNode.get("field").as(...)() 来访问
3. `AvroDeserializationSchema`，序列化Avro格式的数据

如果在反序列化的过程中遇到一个损坏的message，有两种处理办法

1. 在deserialize方法内部抛出异常，这会导致job失败，并且重启
2. 返回null，忽略损坏message

需要注意的是，由于consumer's的容错机制，失败的job会导致重新comsumer重新进行反序列化，从而导致死循环

## Kafka Comsumer的偏移量配置

```java
// 获得stream的执行环境
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

FlinkKakfaComsumer08<String> myComsumer = new FlinkKafkaComsumer08<>(...);

myConsumer.setStartFromEarliest();     // start from the earliest record possible
myConsumer.setStartFromLatest();       // start from the latest record
myConsumer.setStartFromTimestamp(...); // start from specified epoch timestamp (milliseconds)
myConsumer.setStartFromGroupOffsets(); // the default behaviour

DataStream<String> stream = env.addSource(myComsumer);

```




## Reference

[Apache Kafka Connector](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/connectors/kafka.html)
