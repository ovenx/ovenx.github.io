---
title: let's encrypt 小记
date: 2017-10-12 08:00:00
categories: ["程序世界"]
tags:
- ssh
- let's encrypt
---

本文的环境是 nginx1.3 + centos7，nginx 设置的根目录 /www


### 生成 Diffie-Hellman Parameters
生成这个文件的目的是加强 ssl 的安全性。 当然这一步不是必需的，但是如果没有这一步，网站的 ssl [评级](https://www.ssllabs.com/ssltest/)将无法到达 `A+`。
```bash
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

### 配置 nginx ssl
添加文件 `/etc/nginx/default.d/ssl.conf` 这里参考 [https://cipherli.st/](https://cipherli.st/)。 

```bash
ssl_protocols TLSv1.2;# Requires nginx >= 1.13.0 else use TLSv1.2
ssl_prefer_server_ciphers on;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:DHE-RSA-AES256-SHA;
ssl_session_timeout  10m;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off; # Requires nginx >= 1.5.9
ssl_stapling on; # Requires nginx >= 1.3.7
ssl_stapling_verify on; # Requires nginx => 1.3.7
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
add_header X-Robots-Tag none;
```
如果上一步生成了 `dhparam.pem`，需要加上
```bash
ssl_dhparam /etc/ssl/certs/dhparam.pem;
```

添加文件 `/etc/nginx/default.d/letsencrypt.conf`
```bash
location /.well-known/acme-challenge {
        root /www/letsencrypt;
}
```

创建 ssl 认证的目录
```bash
sudo mkdir -p /www/letsencrypt/.well-known/acme-challenge
```
### 配置 nginx http
这一步主要是为了在生成证书时，验证 `/www/letsencrypt`
```bash
server {
    server_name www.domain_1.com;
    listen 80;
    listen [::]:80;
    include /etc/nginx/default.d/letsencrypt.conf;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

### 安装 certbot
```bash
sudo yum install certbot
```
### 使用 certbot 生成证书

```bash
certbot certonly --cert-name domain.com --webroot -w /www/letsencrypt -d www.domain_1.com -d wwww.domain_2.com
```


### 配置 nginx https

添加文件 `/etc/nginx/config.d/domain_1.conf`，其中指定了上一步生成的证书位置

```bash
server {
    server_name  www.domain_1.com;
    listen 443 http2 ssl default_server;
    listen [::]:443 http2 ssl default_server;
    ssl_certificate /etc/letsencrypt/live/www.domain_1.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.domain_1.com/privkey.pem;
    include /etc/nginx/default.d/ssl.conf;

    root /www;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}

# redirect to https
server {
    server_name www.domain_1.com;
    listen 80;
    listen [::]:80;
    include /etc/nginx/default.d/letsencrypt.conf;
    location / {
        return 301 https://$host$request_uri;
    }
}
```



### 自动更新证书
Certbot 可以更新 30 天内期限的证书，测试更新可以使用
```bash
certbot renew --dry-run
```
更方便的做法是设置 crontab 来自动更新证书，先编写一个脚本`/root/letsencrypt.sh`
```bash
#!/bin/bash
systemctl reload nginx
```
然后编写 crontab，certbot 更新完成后会自动完成调用 `letsencrypt.sh` 重启 nginx
```bash
20 03 * * * certbot renew --quiet --deploy-hook /root/letsencrypt.sh
```