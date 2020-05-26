# 4.4 Nginx

[TOC]

## 安装

```bash
yum install -y nginx
```

## 配置文件夹

`/etc/nginx`

## 配置示例

```nginx
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user www;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name www.dalongm.top @.dalongm.top;
        rewrite ^(.*)$ https://$host$1 permanent;
    }

    server {
        listen 443 default_server ssl;
        listen [::]:443 default_server ssl;
        server_name www.dalongm.top @.dalongm.top;
        ssl_certificate cert/3961127_dalongm.top.pem; #将domain name.pem替换成您证书的文件名。
        ssl_certificate_key cert/3961127_dalongm.top.key; #将domain name.key替换成您证书的密钥文件名。
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #使用此加密套件。
        ssl_protocols TLSv1.2; #使用该协议进行配置。
        ssl_prefer_server_ciphers on;

        location / {
            proxy_pass https://127.0.0.1:9443;
            proxy_set_header Host $host;
        }

        error_page 404 /404.html;
        location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }

    server {
        listen 80;
        listen [::]:80;
        server_name score.dalongm.top;
        rewrite ^(.*)$ https://$host$1 permanent;
    }

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        server_name score.dalongm.top;
        ssl_certificate cert/3961362_score.dalongm.top.pem; #将domain name.pem替换成您证书的文件名。
        ssl_certificate_key cert/3961362_score.dalongm.top.key; #将domain name.key替换成您证书的密钥文件名。
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #使用此加密套件。
        ssl_protocols TLSv1.2; #使用该协议进行配置。
        ssl_prefer_server_ciphers on;

        location / {
            proxy_pass https://127.0.0.1:9200;
            proxy_set_header Host $host;
        }

        error_page 404 /404.html;
        location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
```

