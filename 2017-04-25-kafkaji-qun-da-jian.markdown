---
layout: post
title: "Kafka集群搭建"
date: 2017-04-25 00:58:41 +0800
comments: true
categories: Dev
tags: [Kafka]
---
参考:[kafka环境搭建2-broker集群+zookeeper集群](http://www.jianshu.com/p/dc4770fc34b6)

<!--more-->

###首先搭建zk集群
前面已经介绍过了
### Kafka配置修改
修改`config/server.properties`

```
broker.id=1 
host.name=192.168.138.128
log.dirs=/root/kafka-logs
zookeeper.connect=192.168.138.128:2181,192.168.138.129:2181,192.168.138.130:2181
```
其中broker.id 根据集群机器分别编号1,2,3...

### 启动zk

```
bin/kafka-server-start.sh -daemon config/server.properties
```

### 创建kafka topic节点

```
[root@master logs]# ../bin/kafka-topics.sh --describe --zookeeper 192.168.138.128:2181 --topic topic_test
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
Topic:topic_test	PartitionCount:3	ReplicationFactor:3	Configs:
	Topic: topic_test	Partition: 0	Leader: 2	Replicas: 2,3,1	Isr: 2,1,3
	Topic: topic_test	Partition: 1	Leader: 3	Replicas: 3,1,2	Isr: 1,2,3
	Topic: topic_test	Partition: 2	Leader: 1	Replicas: 1,2,3	Isr: 2,1,3
```

### 查询kafka topic状态
```
bin/kafka-topics.sh --describe --zookeeper 192.168.138.129:2181 --topic topic_test
```

### Kafka消费者
```
bin/kafka-console-consumer.sh --bootstrap-server 192.168.138.128:9092 --topic topic_test --from-beginning
```

### Kafka生产者
```
bin/kafka-console-producer.sh --broker-list 192.168.138.130:9092 --topic topic_test --from-beginning
```

### Kafka验证消息
```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 192.168.138.128:9092,192.168.138.129:9092,192.168.138.130:9092 --topic topic_test --time -1
```

###查看消费者Offset
```
[root@master kafka_2.12-0.10.2.0]# bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list 192.168.138.128:9092,192.168.138.129:9092,192.168.138.130:9092 --topic topic_test --time -1
topic_test:2:4
topic_test:1:5
topic_test:0:4
```
