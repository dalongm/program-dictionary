# 第四章 MySQL

[TOC]



## 安装MySQL

### 卸载mariadb

```shell
rpm -qa | grep mariadb
rpm -e --nodeps mariadb-libs
```

### 在线安装

```shell
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
sudo yum -y install mysql-server
```

### rpm安装

```shell
rpm -ivh mysql-community-common
rpm -ivh mysql-community-lib
rpm -ivh mysql-community-client
rpm -ivh mysql-community-server

service mysqld start
```

## 查看安装默认密码

```shell
cat /var/log/mysqld.log |grep password
```

## 修改密码

```shell
mysql > set global validate_password_policy=0;
mysql > set global validate_password_length=1;
mysql > ALTER USER USER() IDENTIFIED BY '123456';
```

```shell
mysql > use mysql;
mysql > update user set password=password('123456') where user='root';
mysql > select host,user, password from user;
mysql > GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "root";
mysql > flush privileges;
mysql > exit;
service mysqld restart
```

> windows mysql 5.7 修改密码

```pow
mysql > use mysql;
mysql > update user set authentication_string=password('新密码') where user='root' and Host='localhost';
mysql > flush privileges;
mysql > exit;
```

## 允许远程

```shell
mysql > use mysql;
mysql > GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "新密码";
mysql > flush privileges;
mysql > exit;
service mysqld restart
```

## 开放端口

```shell
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
iptables -L -n
```

## 查看连接占用

```mysql
show processlist;
show full processlist;
show status;
```

