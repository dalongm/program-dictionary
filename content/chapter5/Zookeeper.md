# 5.6 Zookeeper

[TOC]

## 安装

```shell
#!/bin/bash

BASE_DIR=$(readlink -f $(dirname $0))
cd ${BASE_DIR}

INSTALL_PATH=/usr/local

tar -zxf apache-zookeeper-3.6.0-bin.tar.gz
rm -rf ${INSTALL_PATH}/apache-zookeeper-3.6.0-bin
rm -rf ${INSTALL_PATH}/zookeeper
mv apache-zookeeper-3.6.0-bin ${INSTALL_PATH}/
ln -s ${INSTALL_PATH}/apache-zookeeper-3.6.0-bin ${INSTALL_PATH}/zookeeper
cp ${INSTALL_PATH}/zookeeper/conf/zoo_sample.cfg ${INSTALL_PATH}/zookeeper/conf/zoo.cfg
```

## 系统服务

### 配置文件zookeeper.service

```properties
[Unit]
Description=Apache ZooKeeper highly reliable distributed coordination
After=network.target

[Service]
Type=simple
User=root
Group=root
Restart=always
RestartSec=1
ExecStart=/usr/local/zookeeper/bin/zkServer.sh start-foreground
ExecStop=/usr/local/zookeeper/bin/zkServer.sh stop

[Install]
WantedBy=multi-user.target
```

### 添加自启与启动

```shell
#!/bin/bash

BASE_DIR=$(readlink -f $(dirname $0))
cd ${BASE_DIR}

INSTALL_PATH=/usr/local

${INSTALL_PATH}/zookeeper/bin/zkServer.sh stop
rm -f /usr/lib/systemd/system/zookeeper.service
cp zookeeper.service /usr/lib/systemd/system
systemctl daemon-reload
systemctl enable zookeeper.service
systemctl stop zookeeper.service
systemctl start zookeeper.service
```

