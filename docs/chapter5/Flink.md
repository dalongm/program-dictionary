# 5.7 Flink

[TOC]

## 安装

```shell
#!/bin/bash

BASE_DIR=$(readlink -f $(dirname $0))
cd ${BASE_DIR}

INSTALL_PATH=/usr/local

tar -zxf flink-1.9.2-bin-scala_2.11.tgz
DIR=flink-1.9.2

rm -rf ${INSTALL_PATH}/flink-1.9.2
rm -rf ${INSTALL_PATH}/flink
mv ${DIR}  ${INSTALL_PATH}
chown -R root ${INSTALL_PATH}/${DIR}
chgrp -R root ${INSTALL_PATH}/${DIR}
ln -s ${INSTALL_PATH}/${DIR} ${INSTALL_PATH}/flink
```

## 启动

```bash
${INSTALL_PATH}/flink/bin/stop-cluster.sh
```

## 停止

```bash
${INSTALL_PATH}/flink/bin/start-cluster.sh
```

