# 3.1 MySQL

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

##  ON DUPLICATE KEY UPDATE 

> 判断记录是否存在,不存在则插入存在则更新的场景.

举个例子，字段a被定义为UNIQUE，并且原数据库表table中已存在记录(2,2,9)和(3,2,1)，如果插入记录的a值与原有记录重复，则更新原有记录，否则插入新行：

```mysql
INSERT INTO TABLE (a,b,c) VALUES 
(1,2,3),
(2,5,7),
(3,3,6),
(4,8,2)
ON DUPLICATE KEY UPDATE b=VALUES(b);
```

 以上SQL语句的执行，发现(2,5,7)中的a与原有记录(2,2,9)发生唯一值冲突，则执行ON DUPLICATE KEY UPDATE，将原有记录(2,2,9)更新成(2,5,9)，将(3,2,1)更新成(3,3,1)，插入新记录(1,2,3)和(4,8,2)
注意：ON DUPLICATE KEY UPDATE只是MySQL的特有语法，并不是SQL标准语法！ 

## 删除字段为NULL

```mysql
DELETE FROM table_name WHERE type is NULL;
```

## 查看连接

```mysql
show full processlist;
```

## 查看状态

```mysql
show status;
```

