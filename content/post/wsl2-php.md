---
title: WSL2 下配置 PHP 环境
date: 2020-06-23T18:24:00+08:00
categories: ["程序世界"]
tags: ["wsl", "php", "debian"]
---

## 系统

* wsl2
* debian 10
* windows 10 专业版 1909

## 设置默认 root 登录

```bash
debian config --default-user root
```

## 配置国内源

```bash
cp /etc/apt/source.list /etc/apt/source.list.bak
apt install apt-transport-https ca-certificates
vi source.list

# 复制下面内容到 source.list
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
```

> <https://mirror.tuna.tsinghua.edu.cn/help/debian/>

## 更新源

```bash
apt update
apt upgrade
```

## 安装 nginx

```bash
apt-get install nginx
service nginx start # 启动nginx
```

## 安装 php

### 下载 GPG key

```bash
apt install lsb-release apt-transport-https ca-certificates
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
```

### 添加 SURY

```bash
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
```

### 更新仓库

```bash
apt update
```

### 安装 php 及扩展

```bash
# for apache
# apt install -y apache2 libapache2-mod-php7.4
apt install -y php7.4-fpm php7.4-mysql php7.4-curl php7.4-gd php7.4-mbstring php7.4-xml php7.4-xmlrpc php7.4-zip php7.4-opcache php7.4-dev php7.4-bcmath
service php7.4-fpm start
```

> <https://computingforgeeks.com/how-to-install-latest-php-on-debian/>

### 添加 nginx 站点配置

```bash
server {

        listen 80;
        listen [::]:80;

        server_name example.test;

        root /var/www/html;
        index index.php index.html;

        location / {
                try_files $uri $uri/ =404;
                # 跨域处理
                #if ($request_method = 'OPTIONS' ) {
                #      add_header Access-Control-Allow-Origin *;
                #      add_header Access-Control-Allow-Headers Authorization,Content-Type,Accept,Origin,User-Agent,DNT,Cache-Control,X-Mx-ReqToken,X-Data-Type,X-Requested-With;
                #      add_header Access-Control-Allow-Methods GET,POST,OPTIONS,HEAD,PUT;
                #      add_header Access-Control-Allow-Credentials true;
                #      return 204;
                #}
        }

        location ~* \.php$ {
            fastcgi_pass unix:/run/php/php7.4-fpm.sock;
            include         fastcgi_params;
            fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
            fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
        }

}
service nginx restart
```

### 关于 wsl2 运行 php-fpm 很慢的问题

在 nginx.conf 的 http 块中添加

```bash
fastcgi_buffering off;
```

> <https://stackoverflow.com/questions/46286420/php7-0-fpm-extremly-slow-on-ubuntu-windows-subsystem-linux>

## 安装 redis

安装 redis

```bash
apt-get install redis-server
apt-get install php-redis
```

设置权限

```bash
sudo chown redis:redis /var/run/redis/redis-server.pid
sudo chmod 644 /var/run/redis/redis-server.pid
```

启动 redis

```bash
service redis-server start
```

## 安装 swoole

下载安装包

```bash
wget https://github.com/swoole/swoole-src/archive/v4.5.2.tar.gz
tar -zxvf v4.5.2.tar.gz

```

编译

```bash
cd swoole-src-4.5.2
/usr/bin/phpize7.4
./configure
make && sudo make install
```

修改 php.ini，/etc/php/7.4/fpm/php.ini

```bash
extension=swoole.so
```

重启 php-fpm

```bash
service php7.4-fpm restart
```

## wsl 自启动服务

创建开机脚本 /root/init.wsl

```bash
#! /bin/sh
/etc/init.d/nginx start
/etc/init.d/php7.4-fpm start
/etc/init.d/redis-server start
```

添加权限

```bash
chmod +x /root/init.wsl
```

windwos 下编写 vbs 文件 start-wsl.vbs

```bash
Set ws = WScript.CreateObject("WScript.Shell")
ws.run "wsl -d debian -u root /root/init.wsl"
```

加入 windows 启动项

```bash
win+R
shell:startup
```

将 start-wsl.vbs 移动在启动文件夹中

> <https://zhuanlan.zhihu.com/p/74521796>
> 