# Java与Osquery证书

[TOC]

## 目的

在使用`Java`采集`Osquery`端数据时，因为`TLS`证书问题困扰了很久，一直出现` Request error: certificate verify failed`，详情如下：

```bash
I0828 14:19:27.860253 30918 database.cpp:563] Checking database version for migration
W0828 14:19:32.998385 30918 init.cpp:686] Error reading config: Request error: certificate verify failed
I0828 14:19:33.026228 30918 events.cpp:863] Event publisher not enabled: auditeventpublisher: Publisher disabled via configuration
I0828 14:19:33.026298 30918 events.cpp:863] Event publisher not enabled: syslog: Publisher disabled via configuration
```

未找到相关的问题博文说明，于是整理此文。

## 适用范围

支持所有的`Osquery`版本，高于`4.1.2`版本会验证`名字与姓氏`是否与服务端的`IP`或`主机名`一致，不一致则报错，所以在生成证书时需要指定`名字与姓氏`为服务端的`IP`或`主机名`。

> [已解决] 初步测试，支持`Osquery <= 4.1.2`。高版本会报错，推测更改了`TLS`验证方式。

## 生成KeyStore文件

### 交互生成

```bash
keytool -genkey -validity 36600 -alias www.test.cn -keyalg RSA -keystore test.keystore
```

其中：
-genkey表示生成密钥
-validity指定证书有效期，这里是36600天
-alias指定别名，这里是www.test.cn
-keyalg指定算法，这里是RSA
-keystore指定存储位置，这里是test.keystore

**控制台输出：**

```bash
输入keystore密码：  
再次输入新密码:  
您的名字与姓氏是什么？  
  [Unknown]：  127.0.0.1  
您的组织单位名称是什么？  
  [Unknown]：  
您的组织名称是什么？  
  [Unknown]：    
您所在的城市或区域名称是什么？  
  [Unknown]：    
您所在的州或省份名称是什么？  
  [Unknown]：    
该单位的两字母国家代码是什么  
  [Unknown]：    
CN=127.0.0.1, OU=, O=, L=, ST=, C= 正确吗？  
  [否]：  Y  
```

这儿的密码为：123456

在命令执行的同目录下就会生成一个`test.keystore`文件。

### 一次生成

```bash
keytool -genkey -alias www.test.cn -keypass 123456 -keyalg RSA -keysize 1024 -validity 36600 -keystore  test.keystore -storepass 123456 -dname "CN=127.0.0.1, OU=, O=, L=, ST=, C="
```



### 查看信息

```bash
keytool -list  -v -keystore test.keystore -storepass 123456
```



## 获取自签名证书

光有KeyStore文件是不够的，还需要证书文件，证书才是直接提供给外界使用的公钥凭证。

有两种方式，一种是已有keystore文件，使用`keytool`生成；另一种是没有keystore文件，使用`openssl`请求获取。

### keytool创建公钥凭证

#### 交互生成

```bash
keytool -export -keystore test.keystore -alias www.test.cn -file test.cer -rfc
```

#### 一次生成

```bash
keytool -export -keystore test.keystore -storepass 123456 -alias www.test.cn -file test.cer -rfc
```



查看公钥凭证

```bash
cat test.cer
```

>```
>-----BEGIN CERTIFICATE-----
>MIIDlzCCAn+gAwIBAgIULTkYO65dkEg2o0mHM6ZMqWsK5w4wDQYJKoZIhvcNAQEL
>BQAwWzELMAkGA1UEBhMCVVMxEzARBgNVBAgMCkNhbGlmb3JuaWExGDAWBgNVBAoM
>D29zcXVlcnktdGVzdGluZzEdMBsGA1UEAwwUb3NxdWVyeS11bml0dGVzdHMtY2Ew
>HhcNMjAwMjExMjEzMzMzWhcNMzAwMjExMjEzMzMzWjBbMQswCQYDVQQGEwJVUzET
>MBEGA1UECAwKQ2FsaWZvcm5pYTEYMBYGA1UECgwPb3NxdWVyeS10ZXN0aW5nMR0w
>ggEPADCCAQoCggEBAOKMqGZCm+QuTu+QjNCG3cl42g763RBmp9Xn3SzIPinUgbWz
>nNBdgzlwkjmRBnevuIEMkLFSSVyH9J44sLj/5vBvwq0upJZv/mZwwb+fBkPlek1n
>KRqoPZs/smqVWLQjz6vWsxqS5DKyzd1kyv+hBOkPkybpYlMFC9UyhPkYWAwJ95TL
>4Ag69qCymsGLqa+Ii/lpo5JJ4t41quBZYXqogz01N5c/w/hifKTx0ba/UTW/zMls
>xsugmwZsMMn/uIoFg23CIns38RHzBQEkyDHHt8cCAwEAAaNTMFEwHQYDVR0OBBYE
>FMhAbnhX45+2kR+Ch/idK9dNB1kAMB8GA1UdIwQYMBaAFMhAbnhX45+2kR+Ch/id
>K9dNB1kAMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAMrFOgO3
>x9CdWIjKYB2LaA46EEEzLu/2fqfqoIutGVV1g0/WZ4m3roS2d1w+qFemGIAXbypf
>yQj2kLpu6++3xvHylWgpa3IoQimZs0x6Kafqe6Gz4iEKg4lSmFXR1zLhXJeX9QMg
>+HqTZMlDoen4K+Gd+kYA5ky2t7auS5mHYa3pL0m3TuKUTO6+Y5WH8SEdo4bcNsUk
>+D9AmV9Nsmm91mE6xgf+sFxlQpf7bZsFhqceIiApUujR3MSpq9lHC1fdVkaD6ViV
>4Lxx7odIhaxQ2NQ32qYDqn3Ieu7vdUdgVry++CzGsv342IMU7FKuYAWD3GFo+z9Y
>s3oPBKzj3Spr7H4=
>-----END CERTIFICATE-----
>```

### OPENSSL连接获取公钥凭证

```bash
openssl s_client -connect ${hostname}:${port}

# 例如
openssl s_client -connect 192.168.1.100:8080
```

> ```
> CONNECTED(00000114)
> Can't use SSL_get_servername
> depth=0 C = US, ST = California, O = osquery-testing, CN = osquery-unittests-ca
> verify error:num=18:self signed certificate
> verify return:1
> depth=0 C = US, ST = California, O = osquery-testing, CN = osquery-unittests-ca
> verify return:1
> ---
> Certificate chain
>  0 s:C = US, ST = California, O = osquery-testing, CN = osquery-unittests-ca
>    i:C = US, ST = California, O = osquery-testing, CN = osquery-unittests-ca
> ---
> Server certificate
> -----BEGIN CERTIFICATE-----
> MIIDlzCCAn+gAwIBAgIULTkYO65dkEg2o0mHM6ZMqWsK5w4wDQYJKoZIhvcNAQEL
> BQAwWzELMAkGA1UEBhMCVVMxEzARBgNVBAgMCkNhbGlmb3JuaWExGDAWBgNVBAoM
> D29zcXVlcnktdGVzdGluZzEdMBsGA1UEAwwUb3NxdWVyeS11bml0dGVzdHMtY2Ew
> HhcNMjAwMjExMjEzMzMzWhcNMzAwMjExMjEzMzMzWjBbMQswCQYDVQQGEwJVUzET
> MBEGA1UECAwKQ2FsaWZvcm5pYTEYMBYGA1UECgwPb3NxdWVyeS10ZXN0aW5nMR0w
> ggEPADCCAQoCggEBAOKMqGZCm+QuTu+QjNCG3cl42g763RBmp9Xn3SzIPinUgbWz
> nNBdgzlwkjmRBnevuIEMkLFSSVyH9J44sLj/5vBvwq0upJZv/mZwwb+fBkPlek1n
> KRqoPZs/smqVWLQjz6vWsxqS5DKyzd1kyv+hBOkPkybpYlMFC9UyhPkYWAwJ95TL
> 4Ag69qCymsGLqa+Ii/lpo5JJ4t41quBZYXqogz01N5c/w/hifKTx0ba/UTW/zMls
> xsugmwZsMMn/uIoFg23CIns38RHzBQEkyDHHt8cCAwEAAaNTMFEwHQYDVR0OBBYE
> FMhAbnhX45+2kR+Ch/idK9dNB1kAMB8GA1UdIwQYMBaAFMhAbnhX45+2kR+Ch/id
> K9dNB1kAMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAMrFOgO3
> x9CdWIjKYB2LaA46EEEzLu/2fqfqoIutGVV1g0/WZ4m3roS2d1w+qFemGIAXbypf
> yQj2kLpu6++3xvHylWgpa3IoQimZs0x6Kafqe6Gz4iEKg4lSmFXR1zLhXJeX9QMg
> +HqTZMlDoen4K+Gd+kYA5ky2t7auS5mHYa3pL0m3TuKUTO6+Y5WH8SEdo4bcNsUk
> +D9AmV9Nsmm91mE6xgf+sFxlQpf7bZsFhqceIiApUujR3MSpq9lHC1fdVkaD6ViV
> 4Lxx7odIhaxQ2NQ32qYDqn3Ieu7vdUdgVry++CzGsv342IMU7FKuYAWD3GFo+z9Y
> s3oPBKzj3Spr7H4=
> -----END CERTIFICATE-----
> subject=C = US, ST = California, O = osquery-testing, CN = osquery-unittests-ca
> 
> issuer=C = US, ST = California, O = osquery-testing, CN = osquery-unittests-ca
> 
> ---
> No client certificate CA names sent
> Peer signing digest: SHA256
> Peer signature type: RSA
> Server Temp Key: ECDH, P-256, 256 bits
> ---
> SSL handshake has read 1411 bytes and written 419 bytes
> Verification error: self signed certificate
> ---
> New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
> Server public key is 2048 bit
> Secure Renegotiation IS supported
> Compression: NONE
> Expansion: NONE
> No ALPN negotiated
> SSL-Session:
>     Protocol  : TLSv1.2
>     Cipher    : ECDHE-RSA-AES256-GCM-SHA384
>     Session-ID: 5F4869F53DD054F1EF3E9CBF98F09693FE41CA3AA29F1B06FF2F4C2A88B084B1
>     Session-ID-ctx:
>     Master-Key: 2C4C6F5D9DF2072163AC516522A52505EE111CB19EC912201BED6FD6155B312B670F43EDCBC6732DB32BCB2C50C5EFFA
>     PSK identity: None
>     PSK identity hint: None
>     SRP username: None
>     Start Time: 1598581214
>     Timeout   : 7200 (sec)
>     Verify return code: 18 (self signed certificate)
>     Extended master secret: yes
> ---
> ```

将`-----BEGIN CERTIFICATE-----`以及`-----END CERTIFICATE-----`包括的内容保存到文件`test.cer`中，即

```
-----BEGIN CERTIFICATE-----
MIIDlzCCAn+gAwIBAgIULTkYO65dkEg2o0mHM6ZMqWsK5w4wDQYJKoZIhvcNAQEL
BQAwWzELMAkGA1UEBhMCVVMxEzARBgNVBAgMCkNhbGlmb3JuaWExGDAWBgNVBAoM
D29zcXVlcnktdGVzdGluZzEdMBsGA1UEAwwUb3NxdWVyeS11bml0dGVzdHMtY2Ew
HhcNMjAwMjExMjEzMzMzWhcNMzAwMjExMjEzMzMzWjBbMQswCQYDVQQGEwJVUzET
MBEGA1UECAwKQ2FsaWZvcm5pYTEYMBYGA1UECgwPb3NxdWVyeS10ZXN0aW5nMR0w
ggEPADCCAQoCggEBAOKMqGZCm+QuTu+QjNCG3cl42g763RBmp9Xn3SzIPinUgbWz
nNBdgzlwkjmRBnevuIEMkLFSSVyH9J44sLj/5vBvwq0upJZv/mZwwb+fBkPlek1n
KRqoPZs/smqVWLQjz6vWsxqS5DKyzd1kyv+hBOkPkybpYlMFC9UyhPkYWAwJ95TL
4Ag69qCymsGLqa+Ii/lpo5JJ4t41quBZYXqogz01N5c/w/hifKTx0ba/UTW/zMls
xsugmwZsMMn/uIoFg23CIns38RHzBQEkyDHHt8cCAwEAAaNTMFEwHQYDVR0OBBYE
FMhAbnhX45+2kR+Ch/idK9dNB1kAMB8GA1UdIwQYMBaAFMhAbnhX45+2kR+Ch/id
K9dNB1kAMA8GA1UdEwEB/wQFMAMBAf8wDQYJKoZIhvcNAQELBQADggEBAMrFOgO3
x9CdWIjKYB2LaA46EEEzLu/2fqfqoIutGVV1g0/WZ4m3roS2d1w+qFemGIAXbypf
yQj2kLpu6++3xvHylWgpa3IoQimZs0x6Kafqe6Gz4iEKg4lSmFXR1zLhXJeX9QMg
+HqTZMlDoen4K+Gd+kYA5ky2t7auS5mHYa3pL0m3TuKUTO6+Y5WH8SEdo4bcNsUk
+D9AmV9Nsmm91mE6xgf+sFxlQpf7bZsFhqceIiApUujR3MSpq9lHC1fdVkaD6ViV
4Lxx7odIhaxQ2NQ32qYDqn3Ieu7vdUdgVry++CzGsv342IMU7FKuYAWD3GFo+z9Y
s3oPBKzj3Spr7H4=
-----END CERTIFICATE-----
```

## Spring Boot使用KeyStore

将`test.keystore`放于`Spring Boot`工程的`src/main/resources`目录下，修改Java的Spring配置文件`application.yml`（`application.properties`同理）。

```yaml
server:
  port: 8080
  ssl:
    key-store: classpath:test.keystore
    key-store-type: JKS
    key-store-password: 123456
```

## Osquery使用公钥凭证

将`test.cer`复制到`osquery`所在的服务器，`osqueryd`启动参数`tls_server_certs`指向改文件。

在官网中TLS公钥凭证后缀为`pem`，实际上这儿可以直接将`cer`后缀修改为`pem`。

例如：

```bash
sudo osqueryd \
    --enroll_secret_path=enroll_secret.txt \
    --tls_server_certs=test.cer \
    --tls_hostname=192.168.1.100:8080 \
    --host_identifier=uuid \
    --enroll_tls_endpoint=/enroll \
    --config_plugin=tls \
    --config_tls_endpoint=/config \
    --config_tls_refresh=10 \
    --disable_distributed=false \
    --distributed_plugin=tls \
    --distributed_interval=3 \
    --distributed_tls_max_attempts=3 \
    --logger_plugin=tls \
    --logger_tls_endpoint=/log \
    --logger_tls_period=10
```

## 参考资料

**Osquery官网：** https://osquery.io/

**Osquery检测入侵痕迹：** https://evilanne.github.io/2019/02/20/Osquery%E6%A3%80%E6%B5%8B%E5%85%A5%E4%BE%B5%E7%97%95%E8%BF%B9/

**证书生成：** https://blog.csdn.net/liyong313/article/details/84595695

**证书验证失败：**  https://github.com/osquery/osquery/issues/3303