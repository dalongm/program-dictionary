# 4.6 Docker

## 查看ID

```bash
docker inspect --format="{{.Id}}" jenkins
```

## 保存镜像和容器

复制镜像和复制容器都是通过保存为新镜像而进行的。

### 保存镜像

```bash
# 保存镜像(linux)
docker save ID > xxx.tar
docker load < xxx.tar

# windows
docker save ID -o xxx.tar
docker load -i xxx.tar
```

### 保存容器
```bash
# 保存容器(linux)
docker export ID >xxx.tar
# windows
docker export ID -o xxx.tar

docker import xxx.tar containr:v1
docker run -it containr:v1 bash


```

#### 示例

```bash
docker run --name tomcat-analysis -p 18083:8080 -v /usr/web/gaoxi-log:/opt/tomcat/gaoxi-log -it dalongm/tomcat:v1.0 bash
docker run --name tomcat-redis -p 18084:8080 -v /usr/web/gaoxi-log:/opt/tomcat/gaoxi-log -it dalongm/tomcat:v1.0 bash
```



## 修改apt源

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
echo "" > /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/debian jessie main" >> /etc/apt/sources.list
echo "deb http://mirrors.aliyun.com/debian jessie-updates main" >> /etc/apt/sources.list

apt update
apt install libtinfo5


apt install -y vim
```

## Tomcat

```bash
  <role rolename="manager"/>
  <role rolename="manager-gui"/>
  <role rolename="admin"/>
  <role rolename="admin-gui"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>
  <user username="admin" password="jishimen2019" roles="admin-gui,admin,manager-gui,manager,manager-script,manager-jmx,manager-status"/>
```

