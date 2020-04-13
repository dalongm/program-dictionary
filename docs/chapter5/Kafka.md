# 5.1 Kafka

[TOC]

## 启动Zookeeper

```shell
/usr/local/kafka/bin/zookeeper-server-start.sh /usr/local/kafka/config/zookeeper.properties &
```

## 启动Kafka Broker

```shell
/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties &
```

##  查看kafka topics

```shell
/usr/local/kafka/bin/kafka-topics.sh --list --zookeeper localhost:2181
```

## 新建topic

```shell
/usr/local/kafka/bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --zookeeper localhost:2181 --topic ${topic}
```

## 删除kafka topics

```shell
/usr/local/kafka/bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic ${topic}
```

## 产生消息

```shell
/usr/local/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic ${topic}
```

## 接收消息

```shell
/usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic ${topic}
```

## 查看topic信息

```shell
/usr/local/kafka/bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic ${topic}
```

## 重置组消费位置

```shell
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --all-topics --to-earliest --execute
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-group --reset-offsets --topic ${topic} --to-earliest --execute
```

## 无法启动

### 