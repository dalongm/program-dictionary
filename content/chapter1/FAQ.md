# 1.3 FAQ

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

