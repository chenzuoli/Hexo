---
title: kafka命令行基本操作
date: 2018-07-14 11:48:42
tags: kafka
categories: 消息队列
---
这里先介绍下kafka的基本知识及应用场景：它是一个分布式、高吞吐量、容错性好的消息队列，基于生产者、消费者模型来实现消息的生产消费，在我们的应用系统中，可以起到数据流缓存、解耦合、高效的作用。
下面是kafka命令行的基本操作。
<!--more -->
对于kafka集群的安装配置，这里就不讲解了，apache官网有详细配置方案，可参考：<a href="http://kafka.apache.org/documentation/#configuration">apache kafka配置详解</a>
注：本次使用操作的基本环境为kafka 1.1.0、centos 7.4
# 1.后台启动kafka broker
```
./bin/kafka-server-start.sh -daemon config/server.properties
```
# 2.关闭kafka broker：
```
./bin/kafka-server-stop.sh
```
# 3.创建topic：
```
./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 2 --topic test
```
# 4.创建生产者：
```
./bin/kafka-console-producer.sh --broker-list hadoop31:9092,hadoop32:9092,hadoop33:9092 --topic test
```
# 5.创建消费者：
```
./bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```
# 6.查看所有topic：
```
./bin/kafka-topics.sh --list --zookeeper localhost:2181
```
# 7.查看topic状态：
```
./bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
```
# 8.查看topic消费情况：
```
./bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list hadoop31:9092,hadoop32:9092,hadoop33:9092 --topic sparktest --time -1
```
# 9.删除某个topic：
```
./bin/kafka-topics.sh --delete --zookeeper hadoop31:2181,hadoop32:2181,hadoop33:2181 --topic sparktest  注意设置配置参数delete.topic.enable=true，然后重启kafka和zookeeper才可以生效；
```
好了，到这里还没完，后期会持续更新，我会把我工作中使用到的经过测试没问题的kafka知识记录下来分享给大家，如果有什么问题，请大家mail我，希望大家支持。