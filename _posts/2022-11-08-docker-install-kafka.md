---
layout: post
title: 'docker 安装kafka'
description: 'centos下docker安装使用kafka'
categories: [Docker, kafka]
tags: []
image:  /assets/img/blog/docker-kafka.png

related_posts:
  - 
---

- Table of Contents
{:toc .large-only}


### 下载镜像


```shell
docker pull wurstmeister/kafka
docker pull wurstmeister/zookeeper
```

### docker启动zookeeper

```shell
docker run -d \
  --name zookeeper \
  -p 2181:2181 \
  -v /etc/localtime:/etc/localtime \
  wurstmeister/zookeeper
```

### docker启动kafka

|环境变量名称|描述|
|---|-----|
|KAFKA_BROKER_ID|在kafka集群中，每个kafka都有一个BROKER_ID来区分自己|
|KAFKA_ZOOKEEPER_CONNECT|配置zookeeper管理kafka的路径|
|KAFKA_ADVERTISED_LISTENERS|把kafka的地址端口注册给zookeeper|
|KAFKA_LISTENERS|配置kafka的监听端口|

```shell
docker run -d \
    --restart=always \
    --log-driver json-file \
    --log-opt max-size=100m \
    --log-opt max-file=2 \
    --name kafka \
    -p 9092:9092 \
    -e KAFKA_BROKER_ID=0 \
    -e KAFKA_ZOOKEEPER_CONNECT=10.0.4.16:2181/kafka \
    -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.0.4.16:9092 \
    -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
    -v /etc/localtime:/etc/localtime \
    wurstmeister/kafka
```

### 验证kafka

进入容器

```shell
docker exec -it kafka bash
```

进入bin目录 这里根据不同版本而定

```shell
cd /opt/kafka_2.13-2.8.1/bin/
```



### kafka常用命令

`--zookeeper`参数 和上面的 `KAFKA_ZOOKEEPER_CONNECT` 意思一样，zookeeper中存储的kafka路径

#### 创建topic

```shell
./kafka-topics.sh --zookeeper 10.0.4.16:2181/kafka --create --topic test --replication-factor 1 --partitions 3

# 指定消息大小
kafka-topics.sh --zookeeper 10.0.4.16:2181/kafka --alter --topic test --config max.message.bytes=128000
```

#### 查看topic列表

```shell
./kafka-topics.sh --zookeeper 10.0.4.16:2181/kafka --list
```

#### 查看topic详情

```shell
./kafka-topics.sh --zookeeper 10.0.4.16:2181/kafka --describe --topic test
```

#### 创建生产者

```shell
./kafka-console-producer.sh --broker-list 10.0.4.16:9092 --topic test
```

#### 创建消费者

```shell
./kafka-console-consumer.sh --bootstrap-server 10.0.4.16:9092 --topic test --consumer-property group.id=test
```

#### 查看消费者列表

```shell
./kafka-consumer-groups.sh --bootstrap-server 10.0.4.16:9092 --list
```

#### 查看消费者详情

```SHELL
./kafka-consumer-groups.sh --bootstrap-server 10.0.4.16:9092 --describe --group test
```

#### 删除消费者

```shell
./kafka-consumer-groups.sh --bootstrap-server 10.0.4.16:9092 --delete --group test
```

