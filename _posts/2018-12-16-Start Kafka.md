---
layout:     post
title:      Stark Kafka
subtitle:   Kafka的安装以及部署
date:       2018-12-16
author:     Ashton
header-img: img/post-bg-swift2.jpg
catalog: true
tags:
    - Kafka
    - Real-Time application
    - Ditributed System
---

> 本文首次发布于 [Ashton Blog](http://ashtongao.github.io), 作者 [@ashton](http://github.com/ashtongao) ,转载请保留原文链接.

# Kafka 安装

## 一、获取安装包

[获取Kakfa官方安装包](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.1.0/kafka_2.11-2.1.0.tgz)

```
wget http://mirror.bit.edu.cn/apache/kafka/2.1.0/kafka_2.11-2.1.0.tgz

tar -xzf kafka_2.11-2.1.0.tgz

cd kafka_2.11-2.1.0.tgz
```

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216113431.png)

### Centos 安装JAVA

> 如果已安装请忽略

http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216122310.png)

```
sudo yum install java-1.8.0-openjdk
sudo yum install java-1.8.0-openjdk-devel
```



## 二、启动Server

Kafka依赖Zookeeper，Kafka安装包中已经内置了ZooKeepr Server

通过以下命令启动

```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216113704.png)


ZooKeeper启动之后，开始启动Kafka Server

```
bin/kafka-server-start.sh config/server.properties
```

注意一下机器的内存配置，太小的内存配置会导致kafka启动失败

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216114249.png)

## 三、创建一个topic

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test

```

查看创建的topic

```
bin/kafka-topics.sh --list --zookeeper localhost:2181
> test
```

## 四、开始发送消息

```

bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
> this is a message
> this is another message
```

## 五、开始创建一个consumer

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092
> this is a message
> this is another message
```

## 六、设置集群：Setting up a multi-broker cluster

以上都是启动单个broker的流程，而Kafka的威力在于处理集群任务。

首先，我们对brokers配置进行复制，复制另外两个server

```
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties
```

之后，修改另外两个server的配置

```
vi config/server-1.properties
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dirs=/tmp/kafka-logs-1
vi config/server-2.properties
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dirs=/tmp/kafka-logs-2
```

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216174934.png)

重写的三个内容中：
```
broker.id
    在集群中是节点的唯一标识
listeners
    端口不可以重复，因此需要重写
log.dirs
    日志需要对应每个server一份，所以不能重复
```
之后，分别再次开启另外两个Server

```
bin/kafka-server-start.sh config/server-1.properties
bin/kafka-server-start.sh config/server-2.properties
```



创建一个新的topic

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

```
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic

输出：
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0

```

内容|含义
--|--
leader|负责读写partition的node
replicas|负责复制分区log信息的node列表
isr|处于“同步中”（"in-sync"）的节点，

可以看到，broker.id=1的节点现在是leader，对my-replicated-topic发送message
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
```

启动消费者
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
```

测试一下容错能力，关闭leader节点

```
ps aux | grep server-1.properties

找到对应PID，13954

kill -9 13954
```

```
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic

输出信息：
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```

但生产者还是能正常发送信息，leader变成了2

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
```

## 七、从其他输入源获取信息

上面我们都是手动输入信息，但是总不能每次都是通过命令行来操作吧。

所以，我们可以看下kafka是怎么从文件中读取信息

我们启动一个`standalone`模式，其实就是起了一个本地进程，同时穿传入三份配置

```
bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

Kafka的连接进程，包含要连接的kafka代理，以及序列化数据格式
```
config/connect-standalone.properties   
```

以下两个文件都是connector，会拥有一个唯一的名字，而且会对connnector class进行实例化

```
​config/connect-file-source.properties   
config/connect-file-sink.properties
```

​从文件中读取信息，并创建一个topic：`config/connect-file-source.properties`

连通器（sink collector），从kafka topic中读取信息：`config/connect-file-sink.properties`
​    

流程解读：

file collector读取test.txt文件，并且将产生的信息推给 connect-test topic

sink collector读取topic connect-test信息，并将他们写入到test.sink.txt中

```
cat test.sink.txt
foo\nbar
```

由于信息已经保留在 connect-test中，我们可以通过console consumer看到topic的数据

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
```

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216190102.png)

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216185853.png)


持续写入文件

```
echo Another line>> test.txt
```

可以看到console comsumer又多出了一个schema

![](2018-12-16-Start%20Kafka/2018-12-16-Start%20Kafka-20181216190512.png)



## Reference 
[QuickStart](https://kafka.apache.org/quickstart)

