# 5.3 HDFS

[TOC]



## 退出安全模式

```shell
bin/hadoop dfsadmin -safemode leave
```

## 进入安全模式

```shell
bin/hadoop dfsadmin -safemode enter
```

## 恢复edits不一致

```shell
bin/hadoop namenode -recover
```

在HDFS中，提供了fsck命令，用于检查HDFS上文件和目录的健康状态、获取文件的block信息和位置信息等。
fsck命令必须由HDFS超级用户来执行，普通用户无权限。

## 查看文件中损坏的块（-list-corruptfileblocks）

```shell
[root@master sbin]$ hdfs fsck / -list-corruptfileblocks
```

## 将损坏的文件移动至/lost+found目录（-move）

```shell
[root@master sbin]$ hdfs fsck / -move
```

## 删除损坏的文件（-delete）

```shell
[root@master sbin]$ hdfs fsck / -delete
```

## 检查并列出所有文件状态（-files）

```shell
[root@master sbin]$ hdfs fsck / -files
```

## 查看dfs块的报告

```shell
[root@master sbin]$ hdfs dfsadmin -report
```

## 查看目录

```shell
[root@master sbin]$ hdfs -ls /
```

## HDFS datanode无法启动

Directory /data05/block is in an inconsistent state: cluster Id is incompatible with others.

1. 停止namenode

2. 删除data的current内容

    ```shell
     rm -rf /data*/block/current/*
    ```

3. 格式化

    ```shell
    ./hdfs namenode -format
    ```

4. 重启


