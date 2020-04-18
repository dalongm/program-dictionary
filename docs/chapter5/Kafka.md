# 5.1 Kafka

[TOC]

## 安装

```shell
#!/bin/bash

BASE_DIR=$(readlink -f $(dirname $0))
cd ${BASE_DIR}

INSTALL_PATH=/usr/local

tar -zxf kafka_2.11-2.4.1.tgz
rm -rf ${INSTALL_PATH}/kafka_2.11-2.4.1
rm -rf ${INSTALL_PATH}/kafka
mv kafka_2.11-2.4.1  ${INSTALL_PATH}
ln -s ${INSTALL_PATH}/kafka_2.11-2.4.1 ${INSTALL_PATH}/kafka
```

## 系统服务

### 配置文件kafka.service

```properties
[Unit]
Description=Apache Kafka: A Distributed Streaming Platform
After=network.target

[Service]
ExecStart=/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties
ExecStop=/usr/local/kafka/bin/kafka-server-stop.sh

[Install]
WantedBy=multi-user.target
```

### 添加自启与启动

```shell
#!/bin/bash

BASE_DIR=$(readlink -f $(dirname $0))
cd ${BASE_DIR}

INSTALL_PATH=/usr/local

${INSTALL_PATH}/kafka/bin/kafka-server-stop.sh
rm -f /usr/lib/systemd/system/kafka.service
cp kafka.service /usr/lib/systemd/system
systemctl daemon-reload
systemctl enable kafka.service
systemctl stop kafka.service
systemctl start kafka.service
```

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
