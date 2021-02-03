---
title: golang systemd 部署
date: 2021-02-02T18:00:00+08:00
categories: ["程序世界"]
tags: ["golang"]
---

## 新建 systemd 文件

在 /etc/systemd/system/ 目录下新建service  `example.servie`

```bash
[Unit]
Description=example

[Service]
Type=simple
Restart=always
RestartSec=5
ExecStart=/www/example/main        # 可执行文件的路径
ExecReload=/bin/kill -HUP $MAINPID
PIDFile=/var/run/example.pid       # pid file
WorkingDirectory=/www/example

[Install]
WantedBy=multi-user.target
```

重新加载配置文件

```bash
systemctl daemon-reload
```

常用操作

```bash
systemctl start example
systemctl restart example  # 重启
systemctl enable example   # 添加自启动
```


## 配置 nginx 代理

```bash
server {
    server_name  example.com;
    listen 443 http2 ssl;
    listen [::]:443 http2 ssl;
    ssl_certificate /etc/letsencrypt/live/example.cn/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.cn/privkey.pem;
    include /etc/nginx/default.d/ssl.conf;

    root /www/example;
    index index.html;
        location / {
        proxy_pass http://127.0.0.1:4000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

# redirect to https
server {
    server_name example.com;
    listen 80;
    listen [::]:80;
    include /etc/nginx/default.d/letsencrypt.conf;
    location / {
        return 301 https://$host$request_uri;
    }
}

```

## 部署脚本

新建 `deploy.sh`

```bash
#!/bin/sh  
  
echo "Step 1: building ..."  
CGO\_ENABLED\=0 GOOS\=linux GOARCH\=amd64 go build \-ldflags\="-s -w" main.go  
  
echo "Step 2: Uploading ..."  
rsync \-zp --progress main  root@server:/www/example/main
  
echo "Step 3: Restarting ..."  
ssh root@server "systemctl restart example"

```
