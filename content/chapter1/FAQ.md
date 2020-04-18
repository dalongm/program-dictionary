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

