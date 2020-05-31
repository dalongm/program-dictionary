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

## 注册系统服务(standalone模式)

### 配置文件

`/usr/lib/systemd/system/flink.service`

```properties
[Unit]
Description=Apache Flink: Stateful Computations over Data Streams
After=network.target

[Service]
User=root
Group=root
Type=forking
ExecStart=/usr/local/flink/bin/start-cluster.sh
ExecStop=/usr/local/flink/bin/stop-cluster.sh

[Install]
WantedBy=multi-user.target
```

### 开机自启

```bash
#!/bin/bash

BASE_DIR=$(readlink -f $(dirname $0))
cd ${BASE_DIR}

INSTALL_PATH=/usr/local

${INSTALL_PATH}/kafka/bin/kafka-server-stop.sh
rm -f /usr/lib/systemd/system/kafka.service
cp kafka.service /usr/lib/systemd/system

# 重载配置
systemctl daemon-reload
# 启用自启
systemctl enable kafka.service
# 停止
systemctl stop kafka.service
# 启动
systemctl start kafka.service
```

## 启动（使用系统服务代替）

```bash
${INSTALL_PATH}/flink/bin/start-cluster.sh
```

## 停止（使用系统服务代替）

```bash
${INSTALL_PATH}/flink/bin/stop-cluster.sh
```

## 修改日志大小

`${INSTALL_PATH}/flink/conf/log4j.properties`

```bash
################################################################################
#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance
#  with the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
# limitations under the License.
################################################################################

# This affects logging for both user code and Flink
log4j.rootLogger=INFO, file

# Uncomment this if you want to _only_ change Flink's logging
#log4j.logger.org.apache.flink=INFO

# The following lines keep the log level of common libraries/connectors on
# log level INFO. The root logger does not override this. You have to manually
# change the log levels here.
log4j.logger.akka=INFO
log4j.logger.org.apache.kafka=INFO
log4j.logger.org.apache.hadoop=INFO
log4j.logger.org.apache.zookeeper=INFO

# Log all infos in the given file
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.file=${log.file}
# 单个日志文件大小
log4j.appender.file.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.append=true
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p %-60c %x - %m%n

# Suppress the irrelevant (wrong) warnings from the Netty channel handler
log4j.logger.org.apache.flink.shaded.akka.org.jboss.netty.channel.DefaultChannelPipeline=ERROR, file

```

`${INSTALL_PATH}/flink/conf/log4j-cli.properties`

```bash
################################################################################
#  Licensed to the Apache Software Foundation (ASF) under one
#  or more contributor license agreements.  See the NOTICE file
#  distributed with this work for additional information
#  regarding copyright ownership.  The ASF licenses this file
#  to you under the Apache License, Version 2.0 (the
#  "License"); you may not use this file except in compliance
#  with the License.  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
# limitations under the License.
################################################################################

log4j.rootLogger=INFO, file

# Log all infos in the given file
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.file=${log.file}
# 单个日志文件大小
log4j.appender.file.MaxFileSize=128MB
# 保留历史文件个数
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.append=true
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p %-60c %x - %m%n


# Log output from org.apache.flink.yarn to the console. This is used by the
# CliFrontend class when using a per-job YARN cluster.
log4j.logger.org.apache.flink.yarn=INFO, console
log4j.logger.org.apache.flink.yarn.cli.FlinkYarnSessionCli=INFO, console
log4j.logger.org.apache.hadoop=INFO, console

log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss,SSS} %-5p %-60c %x - %m%n

# suppress the warning that hadoop native libraries are not loaded (irrelevant for the client)
log4j.logger.org.apache.hadoop.util.NativeCodeLoader=OFF

# suppress the irrelevant (wrong) warnings from the netty channel handler
log4j.logger.org.apache.flink.shaded.akka.org.jboss.netty.channel.DefaultChannelPipeline=ERROR, file

```

## 访问`Flink Web`

http://localhost:8081

## 提交任务

```bash
./flink <ACTION> [OPTIONS] [ARGUMENTS]

The following actions are available:

Action "run" compiles and runs a program.

  Syntax: run [OPTIONS] <jar-file> <arguments>
  "run" action options:
     -c,--class <classname>               Class with the program entry point
                                          ("main()" method or "getPlan()" method).
                                          Only needed if the JAR file does not
                                          specify the class in its manifest.
     -C,--classpath <url>                 Adds a URL to each user code
                                          classloader  on all nodes in the
                                          cluster. The paths must specify a
                                          protocol (e.g. file://) and be
                                          accessible on all nodes (e.g. by means
                                          of a NFS share). You can use this
                                          option multiple times for specifying
                                          more than one URL. The protocol must
                                          be supported by the {@link
                                          java.net.URLClassLoader}.
     -d,--detached                        If present, runs the job in detached
                                          mode
     -n,--allowNonRestoredState           Allow to skip savepoint state that
                                          cannot be restored. You need to allow
                                          this if you removed an operator from
                                          your program that was part of the
                                          program when the savepoint was
                                          triggered.
     -p,--parallelism <parallelism>       The parallelism with which to run the
                                          program. Optional flag to override the
                                          default value specified in the
                                          configuration.
     -py,--python <python-file>           指定Python作业的入口，依赖的资源文件可以通过
                                          `--pyFiles`进行指定。
     -pyfs,--pyFiles <python-files>       指定Python作业依赖的一些自定义的python文件，
                                          如果有多个文件，可以通过逗号(,)进行分隔。支持
                                          常用的python资源文件，例如(.py/.egg/.zip)。
                                          (例如:--pyFiles file:///tmp/myresource.zip
                                          ,hdfs:///$namenode_address/myresource2.zip)
     -pym,--pyModule <python-module>      指定python程序的运行的模块入口，这个选项必须配合
                                          `--pyFiles`一起使用。
     -q,--sysoutLogging                   If present, suppress logging output to
                                          standard out.
     -s,--fromSavepoint <savepointPath>   Path to a savepoint to restore the job
                                          from (for example
                                          hdfs:///flink/savepoint-1537).
     -sae,--shutdownOnAttachedExit        If the job is submitted in attached
                                          mode, perform a best-effort cluster
                                          shutdown when the CLI is terminated
                                          abruptly, e.g., in response to a user
                                          interrupt, such as typing Ctrl + C.
  Options for yarn-cluster mode:
     -d,--detached                        If present, runs the job in detached
                                          mode
     -m,--jobmanager <arg>                Address of the JobManager (master) to
                                          which to connect. Use this flag to
                                          connect to a different JobManager than
                                          the one specified in the
                                          configuration.
     -sae,--shutdownOnAttachedExit        If the job is submitted in attached
                                          mode, perform a best-effort cluster
                                          shutdown when the CLI is terminated
                                          abruptly, e.g., in response to a user
                                          interrupt, such as typing Ctrl + C.
     -yat,--yarnapplicationType <arg>     Set a custom application type for the
                                          application on YARN
     -yD <property=value>                 use value for given property
     -yd,--yarndetached                   If present, runs the job in detached
                                          mode (deprecated; use non-YARN
                                          specific option instead)
     -yh,--yarnhelp                       Help for the Yarn session CLI.
     -yid,--yarnapplicationId <arg>       Attach to running YARN session
     -yj,--yarnjar <arg>                  Path to Flink jar file
     -yjm,--yarnjobManagerMemory <arg>    Memory for JobManager Container
                                          with optional unit (default: MB)
     -yn,--yarncontainer <arg>            Number of YARN container to allocate
                                          (=Number of Task Managers)
     -ynm,--yarnname <arg>                Set a custom name for the application
                                          on YARN
     -yq,--yarnquery                      Display available YARN resources
                                          (memory, cores)
     -yqu,--yarnqueue <arg>               Specify YARN queue.
     -ys,--yarnslots <arg>                Number of slots per TaskManager
     -yst,--yarnstreaming                 Start Flink in streaming mode
     -yt,--yarnship <arg>                 Ship files in the specified directory
                                          (t for transfer), multiple options are
                                          supported.
     -ytm,--yarntaskManagerMemory <arg>   Memory per TaskManager Container
                                          with optional unit (default: MB)
     -yz,--yarnzookeeperNamespace <arg>   Namespace to create the Zookeeper
                                          sub-paths for high availability mode
     -ynl,--yarnnodeLabel <arg>           Specify YARN node label for
                                          the YARN application
     -z,--zookeeperNamespace <arg>        Namespace to create the Zookeeper
                                          sub-paths for high availability mode

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "info" shows the optimized execution plan of the program (JSON).

  Syntax: info [OPTIONS] <jar-file> <arguments>
  "info" action options:
     -c,--class <classname>           Class with the program entry point ("main()"
                                      method or "getPlan()" method). Only needed
                                      if the JAR file does not specify the class
                                      in its manifest.
     -p,--parallelism <parallelism>   The parallelism with which to run the
                                      program. Optional flag to override the
                                      default value specified in the
                                      configuration.


Action "list" lists running and scheduled programs.

  Syntax: list [OPTIONS]
  "list" action options:
     -r,--running     Show only running programs and their JobIDs
     -s,--scheduled   Show only scheduled programs and their JobIDs
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "stop" stops a running program with a savepoint (streaming jobs only).

  Syntax: stop [OPTIONS] <Job ID>
  "stop" action options:
     -d,--drain                           Send MAX_WATERMARK before taking the
                                          savepoint and stopping the pipelne.
     -p,--savepointPath <savepointPath>   Path to the savepoint (for example
                                          hdfs:///flink/savepoint-1537). If no
                                          directory is specified, the configured
                                          default will be used
                                          ("state.savepoints.dir").
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "cancel" cancels a running program.

  Syntax: cancel [OPTIONS] <Job ID>
  "cancel" action options:
     -s,--withSavepoint <targetDirectory>   **DEPRECATION WARNING**: Cancelling
                                            a job with savepoint is deprecated.
                                            Use "stop" instead.
                                            Trigger savepoint and cancel job.
                                            The target directory is optional. If
                                            no directory is specified, the
                                            configured default directory
                                            (state.savepoints.dir) is used.
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode



Action "savepoint" triggers savepoints for a running job or disposes existing ones.

  Syntax: savepoint [OPTIONS] <Job ID> [<target directory>]
  "savepoint" action options:
     -d,--dispose <arg>       Path of savepoint to dispose.
     -j,--jarfile <jarfile>   Flink program JAR file.
  Options for yarn-cluster mode:
     -m,--jobmanager <arg>            Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -yid,--yarnapplicationId <arg>   Attach to running YARN session
     -z,--zookeeperNamespace <arg>    Namespace to create the Zookeeper
                                      sub-paths for high availability mode

  Options for default mode:
     -m,--jobmanager <arg>           Address of the JobManager (master) to which
                                     to connect. Use this flag to connect to a
                                     different JobManager than the one specified
                                     in the configuration.
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths
                                     for high availability mode
```



### 关于`App`资源分配

使用`yarn-cluster`模式可以通过`yjm`和`ytm`参数指定`yarnjobManagerMemory`、`yarntaskManagerMemory`内存大小，单机集群模式不能在提交任务时指定（因为在启动flink时，已经在`flink-conf.yaml`里分配了这两内存）。