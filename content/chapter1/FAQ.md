# 1.3 系统监控

[TOC]

## service异常

>Redirecting to /bin/systemctl restart mysqld.service
>Failed to restart mysqld.service: Activation of org.freedesktop.systemd1 timed out
>See system logs and 'systemctl status mysqld.service' for details.

造成原因是内存不足导致org.freedesktop.logind和org.freedesktop.systemd模块奔溃。

临时解决方案：

```shell
systemctl daemon-reexec
```

再启动登录服务

```shell
systemctl start systemd-logind
```

## 系统监控

### 磁盘占用

```bash
iotop -oP
```

### 内存

```bash
free -h

# 查看进程占用
cat /proc/【进程id】/status

VmPeak: 11008236 kB
VmSize: 11008140 kB
VmLck:         0 kB
VmPin:         0 kB
VmHWM:   5126716 kB
VmRSS:   5126716 kB
VmData:  9433616 kB
VmStk:       132 kB
VmExe:     22168 kB
VmLib:      3200 kB
VmPTE:     15012 kB
VmSwap:        0 kB
```

## `sar`系统监控

`sar [options] [-A] [-o file] t [n]`

其中：

t为采样间隔，n为采样次数，默认值是1；

-o file表示将命令结果以二进制格式存放在文件中，file 是文件名。

options 为命令行选项，sar命令常用选项如下：

```bash
-A：所有报告的总和

-u：输出[CPU]使用情况的统计信息

-v：输出inode、文件和其他内核表的统计信息

-d：输出每一个块设备的活动信息

-r：输出[内存]和交换空间的统计信息

-b：显示[I/O]和传送速率的统计信息

-a：文件读写情况

-c：输出进程统计信息，每秒创建的进程数

-R：输出内存页面的统计信息

-y：终端设备活动情况

-w：输出系统交换活动信息
```

### 查看网速

```bash
sar -n DEV 1 100 
```

### 