# 5.1 Kafka

[TOC]

以Kafka 2.11版本为例，部分操作不同版本不一样。

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
Type=simple
User=root
Group=root
Restart=always
RestartSec=1
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

## 配置文件

`/usr/local/kafka/config/server.properties`

```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# see kafka.server.KafkaConfig for additional details and defaults

############################# Server Basics #############################

# The id of the broker. This must be set to a unique integer for each broker.
broker.id=0

############################# Socket Server Settings #############################

# The address the socket server listens on. It will get the value returned from
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://localhost:9092

# Hostname and port the broker will advertise to producers and consumers. If not set,
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
advertised.listeners=PLAINTEXT://localhost:9092

# Maps listener names to security protocols, the default is for them to be the same. See the config documentation for more details
#listener.security.protocol.map=PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# The number of threads that the server uses for receiving requests from the network and sending responses to the network
num.network.threads=3

# The number of threads that the server uses for processing requests, which may include disk I/O
num.io.threads=8

# The send buffer (SO_SNDBUF) used by the socket server
socket.send.buffer.bytes=102400

# The receive buffer (SO_RCVBUF) used by the socket server
socket.receive.buffer.bytes=102400

# The maximum size of a request that the socket server will accept (protection against OOM)
socket.request.max.bytes=104857600


############################# Log Basics #############################

# A comma separated list of directories under which to store log files
log.dirs=/tmp/kafka-logs

# The default number of log partitions per topic. More partitions allow greater
# parallelism for consumption, but this will also result in more files across
# the brokers.
num.partitions=1

# The number of threads per data directory to be used for log recovery at startup and flushing at shutdown.
# This value is recommended to be increased for installations with data dirs located in RAID array.
num.recovery.threads.per.data.dir=1

############################# Internal Topic Settings  #############################
# The replication factor for the group metadata internal topics "__consumer_offsets" and "__transaction_state"
# For anything other than development testing, a value greater than 1 is recommended for to ensure availability such as 3.
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

############################# Log Flush Policy #############################

# Messages are immediately written to the filesystem but by default we only fsync() to sync
# the OS cache lazily. The following configurations control the flush of data to disk.
# There are a few important trade-offs here:
#    1. Durability: Unflushed data may be lost if you are not using replication.
#    2. Latency: Very large flush intervals may lead to latency spikes when the flush does occur as there will be a lot of data to flush.
#    3. Throughput: The flush is generally the most expensive operation, and a small flush interval may lead to excessive seeks.
# The settings below allow one to configure the flush policy to flush data after a period of time or
# every N messages (or both). This can be done globally and overridden on a per-topic basis.

# The number of messages to accept before forcing a flush of data to disk
#log.flush.interval.messages=10000

# The maximum amount of time a message can sit in a log before we force a flush
#log.flush.interval.ms=1000

############################# Log Retention Policy #############################

# The following configurations control the disposal of log segments. The policy can
# be set to delete segments after a period of time, or after a given size has accumulated.
# A segment will be deleted whenever *either* of these criteria are met. Deletion always happens
# from the end of the log.

# The minimum age of a log file to be eligible for deletion due to age
log.retention.hours=168

# A size-based retention policy for logs. Segments are pruned from the log unless the remaining
# segments drop below log.retention.bytes. Functions independently of log.retention.hours.
#log.retention.bytes=1073741824

# The maximum size of a log segment file. When this size is reached a new log segment will be created.
log.segment.bytes=1073741824

# The interval at which log segments are checked to see if they can be deleted according
# to the retention policies
log.retention.check.interval.ms=300000

############################# Zookeeper #############################

# Zookeeper connection string (see zookeeper docs for details).
# This is a comma separated host:port pairs, each corresponding to a zk
# server. e.g. "127.0.0.1:3000,127.0.0.1:3001,127.0.0.1:3002".
# You can also append an optional chroot string to the urls to specify the
# root directory for all kafka znodes.
zookeeper.connect=localhost:2181

# Timeout in ms for connecting to zookeeper
zookeeper.connection.timeout.ms=6000


############################# Group Coordinator Settings #############################

# The following configuration specifies the time, in milliseconds, that the GroupCoordinator will delay the initial consumer rebalance.
# The rebalance will be further delayed by the value of group.initial.rebalance.delay.ms as new members join the group, up to a maximum of max.poll.interval.ms.
# The default value for this is 3 seconds.
# We override this to 0 here as it makes for a better out-of-the-box experience for development and testing.
# However, in production environments the default value of 3 seconds is more suitable as this will help to avoid unnecessary, and potentially expensive, rebalances during application startup.
group.initial.rebalance.delay.ms=0
```



## 限制日志文件大小

`/usr/local/kafka/config/log4j.properties`

```properties
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Unspecified loggers and loggers with additivity=true output to server.log and stdout
# Note that INFO only applies to unspecified loggers, the log level of the child logger is used otherwise
log4j.rootLogger=INFO, stdout, kafkaAppender

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=[%d] %p %m (%c)%n

log4j.appender.kafkaAppender=org.apache.log4j.RollingFileAppender
# 单个日志文件大小
log4j.appender.kafkaAppender.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.kafkaAppender.MaxBackupIndex=10
log4j.appender.kafkaAppender.append=true
# log4j.appender.kafkaAppender.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.kafkaAppender.File=${kafka.logs.dir}/server.log
log4j.appender.kafkaAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.kafkaAppender.layout.ConversionPattern=[%d] %p %m (%c)%n

log4j.appender.stateChangeAppender=org.apache.log4j.RollingFileAppender
# 单个日志文件大小
log4j.appender.stateChangeAppender.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.stateChangeAppender.MaxBackupIndex=10
log4j.appender.stateChangeAppender.append=true
# log4j.appender.stateChangeAppender.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.stateChangeAppender.File=${kafka.logs.dir}/state-change.log
log4j.appender.stateChangeAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.stateChangeAppender.layout.ConversionPattern=[%d] %p %m (%c)%n

log4j.appender.requestAppender=org.apache.log4j.RollingFileAppender
# 单个日志文件大小
log4j.appender.requestAppender.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.requestAppender.MaxBackupIndex=10
log4j.appender.requestAppender.append=true
# log4j.appender.requestAppender.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.requestAppender.File=${kafka.logs.dir}/kafka-request.log
log4j.appender.requestAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.requestAppender.layout.ConversionPattern=[%d] %p %m (%c)%n

log4j.appender.cleanerAppender=org.apache.log4j.RollingFileAppender
# 单个日志文件大小
log4j.appender.cleanerAppender.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.cleanerAppender.MaxBackupIndex=10
log4j.appender.cleanerAppender.append=true
# log4j.appender.cleanerAppender.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.cleanerAppender.File=${kafka.logs.dir}/log-cleaner.log
log4j.appender.cleanerAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.cleanerAppender.layout.ConversionPattern=[%d] %p %m (%c)%n

log4j.appender.controllerAppender=org.apache.log4j.RollingFileAppender
# 单个日志文件大小
log4j.appender.controllerAppender.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.controllerAppender.MaxBackupIndex=10
log4j.appender.controllerAppender.append=true
# log4j.appender.controllerAppender.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.controllerAppender.File=${kafka.logs.dir}/controller.log
log4j.appender.controllerAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.controllerAppender.layout.ConversionPattern=[%d] %p %m (%c)%n

log4j.appender.authorizerAppender=org.apache.log4j.RollingFileAppender
# 单个日志文件大小
log4j.appender.authorizerAppender.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.authorizerAppender.MaxBackupIndex=10
log4j.appender.authorizerAppender.append=true
# log4j.appender.authorizerAppender.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.authorizerAppender.File=${kafka.logs.dir}/kafka-authorizer.log
log4j.appender.authorizerAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.authorizerAppender.layout.ConversionPattern=[%d] %p %m (%c)%n

# Change the two lines below to adjust ZK client logging
log4j.logger.org.I0Itec.zkclient.ZkClient=INFO
log4j.logger.org.apache.zookeeper=INFO

# Change the two lines below to adjust the general broker logging level (output to server.log and stdout)
log4j.logger.kafka=INFO
log4j.logger.org.apache.kafka=INFO

# Change to DEBUG or TRACE to enable request logging
log4j.logger.kafka.request.logger=WARN, requestAppender
log4j.additivity.kafka.request.logger=false

# Uncomment the lines below and change log4j.logger.kafka.network.RequestChannel$ to TRACE for additional output
# related to the handling of requests
#log4j.logger.kafka.network.Processor=TRACE, requestAppender
#log4j.logger.kafka.server.KafkaApis=TRACE, requestAppender
#log4j.additivity.kafka.server.KafkaApis=false
log4j.logger.kafka.network.RequestChannel$=WARN, requestAppender
log4j.additivity.kafka.network.RequestChannel$=false

log4j.logger.kafka.controller=TRACE, controllerAppender
log4j.additivity.kafka.controller=false

log4j.logger.kafka.log.LogCleaner=INFO, cleanerAppender
log4j.additivity.kafka.log.LogCleaner=false

log4j.logger.state.change.logger=TRACE, stateChangeAppender
log4j.additivity.state.change.logger=false

# Access denials are logged at INFO level, change to DEBUG to also log allowed accesses
log4j.logger.kafka.authorizer.logger=INFO, authorizerAppender
log4j.additivity.kafka.authorizer.logger=false
```

## 基本操作

### 查看kafka topics

```shell
/usr/local/kafka/bin/kafka-topics.sh --list --zookeeper localhost:2181
```

###  新建topic

```shell
/usr/local/kafka/bin/kafka-topics.sh --create --replication-factor 1 --partitions 1 --zookeeper localhost:2181 --topic ${topic}
```

### 删除kafka topics

```shell
/usr/local/kafka/bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic ${topic}
```

### 产生消息

```shell
/usr/local/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic ${topic}
```

### 接收消息

```shell
/usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic ${topic}
```

### 查看topic信息

```shell
/usr/local/kafka/bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic ${topic}
```

### 重置组消费位置

```shell
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group ${groupname} --reset-offsets --all-topics --to-earliest --execute
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group ${groupname} --reset-offsets --topic ${topic} --to-earliest --execute
```

### 查看所有消费组

```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
```

### 查看消费组消费状态

```bash
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group ${groupname}
```

> TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
> api_access      0          3088921         3102980         14059           -               -               -

TOPIC：该group里消费的topic名称

PARTITION：分区编号

CURRENT-OFFSET：该分区当前消费到的offset

LOG-END-OFFSET：该分区当前latest offset

LAG：消费滞后区间，为LOG-END-OFFSET-CURRENT-OFFSET，具体大小需要看应用消费速度和生产者速度，一般过大则可能出现消费跟不上，需要引起应用注意

CONSUMER-ID：server端给该分区分配的consumer编号

HOST：消费者所在主机

CLIENT-ID：消费者id，一般由应用指定

## 无法启动

### ERROR Shutdown broker because all log dirs in /tmp/kafka-logs have failed

1. 删除`/tmp/kafka-logs`下的`offset`和`consume`相关文件，重启。此方法有可能造成数据丢失。
2. 修改`advertised.listeners=PLAINTEXT://:9092`，重启
3. 修改`log.dirs=/tmp/kafka-logs`，重启

