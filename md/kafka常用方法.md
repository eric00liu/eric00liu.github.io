---
title: kafka常用方法
date: 2018/7/13 20:46:25
tags:
- kafka
- python
---
Kafka是由LinkedIn开发的一个分布式的消息系统，使用Scala编写，它以可水平扩展和高吞吐率而被广泛使用。

## kafka搭建
coming soon

### 通过docker-compose启动storm系统
https://blog.sectong.com/blog/storm_log_stat_openfire.html

### 创建一个topic
```
bin/kafka-topics.sh --create --topic Monitordata -partitions 4 --zookeeper x.x.x.x:2181/kafka --replication-factor 1
```
### 删除一个topic
```
bin/kafka-topics.sh --delete --zookeeper x.x.x.x:2181/kafka  --topic Monitordata
```
### 查看topics
```
bin/kafka-topics.sh --describe --zookeeper x.x.x.x:2181/kafka
```

### 查看consume group id
```
bin/kafka-consumer-groups.sh  --bootstrap-server x.x.x.x:9092 --list
bin/kafka-consumer-groups.sh --bootstrap-server x.x.x.x:9092 --group console-consumer-89280 --describe
```

### 用python写入消息到kafka
写入到strom-test这个topic里消息
一个Python的kafka SDK:  https://github.com/dpkp/kafka-python
```
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers='x.x.x.x:9092')  # broker address
producer.send('strom-test', b'some_message_bytes')

```

### 用python从kafka中消费消息
```
from kafka import TopicPartition
from kafka import KafkaConsumer
consumer = KafkaConsumer(bootstrap_servers='x.x.x.x:9092') # broker address
consumer.assign([TopicPartition('flink-out', 0)])
print next(consumer)  # 阻塞等待一条消息
```



