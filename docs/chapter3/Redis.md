# 3.2 Redis

## 安装Redis

```shell
# 下载
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
# 解压
tar xzf redis-5.0.5.tar.gz
# 编译
cd redis-5.0.5
make

# 将编译文件放于/usr/local/redis下
mkdir /usr/local/redis
cp src/redis-cli /usr/local/redis
cp src/redis-server /usr/local/redis
cp redis.conf /usr/local/redis
```
## 允许远程
编辑配置文件`vim /usr/local/redis/redis.conf`，找到相应行修改：

```shell
# bind 127.0.0.1
protected-mode no

# 取消守护启动，用于开机自启
daemonize no
```

## 自启配置
1. 添加服务脚本`vim /etc/systemd/system/redis.service`：

```shell
[Unit]
Description=Redis Server Manager
After=syslog.target
After=network.target
 
[Service]
# Type=simple
PIDFile=/var/run/redis_6379.pid
ExecStart=/usr/local/redis/redis-server /usr/local/redis/redis.conf
ExecStop=/usr/local/redis/redis-cli shutdown
Restart=always
 
[Install]
WantedBy=multi-user.target
```

2. 添加服务，开机启动：

```shell
systemctl daemon-reload
systemctl start redis.service
systemctl enable redis.service
```

添加`redis-cli`软链接：

```shell
ln -s /usr/local/redis/redis-cli /usr/bin/redis-cli
```

## 开放端口

```shell
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload
iptables -L -n
```

## 基本操作

```shell
# 查看所有key
keys *
# 获取所有配置
config get *
```


## 错误

1. 使用`spring boot `连接`redis`发现`redis`中数据无故无规律清空

    a. 修改`redis service`文件`vim /etc/systemd/system/redis.service`

    b. 修改`redis `配置文件，增加`logfile`路径

    c. 修改系统时区

    然后就未出现当前问题，但是并未确定问题的原因。

