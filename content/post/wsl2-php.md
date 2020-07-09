---
title: WSL2 下配置 PHP 环境
date: 2020-06-23T18:24:00+08:00
categories: ["程序世界"]
tags: ["wsl", "php", "debian"]
---

## 系统

* wsl2
* debian 10
* windows 10 专业版 2004

## 安装 wsl2

需要升级到 win10 19041以上

启用 Windows Subsystem for Linux

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

启用 Virtual Machine Platform

```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

将 WSL 2 设置为默认版本

```bash
wsl --set-default-version 2
```

将发行设置为 WSL 2

```bash
wsl -l -v # 查看发行版使用的 WSL 版本
wsl --set-version debian 2
```

microsoft store 中安装 linux 发行版，这里选择的是 debian

设置默认 root 登录

```bash
debian config --default-user root
```

配置国内源

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
vi sources.list

# 复制下面内容到 source.list
deb http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb http://mirrors.aliyun.com/debian-security buster/updates main
deb-src http://mirrors.aliyun.com/debian-security buster/updates main
deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
```

更新源

```bash
apt update
apt upgrade
```

> <https://docs.microsoft.com/en-us/windows/wsl/install-win10#update-to-wsl-2>

## 安装 php

下载 GPG key

```bash
apt install lsb-release apt-transport-https ca-certificates
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
```

添加 SURY

```bash
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
```

更新仓库

```bash
apt update
```

安装

```bash
# for apache
# apt install -y apache2 libapache2-mod-php7.4
apt install -y php7.4-fpm php7.4-mysql php7.4-curl php7.4-gd php7.4-mbstring php7.4-xml php7.4-xmlrpc php7.4-zip php7.4-opcache php7.4-dev php7.4-bcmath
service php7.4-fpm start
```

> <https://computingforgeeks.com/how-to-install-latest-php-on-debian/>

## 安装 php 常用扩展

### swoole

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

修改 php.ini，/etc/php/7.4/fpm/php.ini，/etc/php/7.4/cli/php.ini

```bash
extension=swoole
```

> 使用 php --ini 来定位到 php.ini 的绝对路径，Loaded Configuration File 一项显示的是加载的 php.ini 文件

重启 php-fpm

```bash
service php7.4-fpm restart
```

### protobuf

安装 protobuf

```bash
pecl install protobuf-3.12.2
```

修改 php.ini

```bash
extension=protobuf
```

## 安装 nginx

```bash
apt-get install nginx
service nginx start # 启动nginx
```

添加 nginx 站点配置

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

> 在 WSL 1 中运行 php-fpm 很慢，可以在 nginx.conf 的 http 块中添加

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

启动 redis

```bash
service redis-server start
```

> 在 WSL 1 中启动时出现 matching on world-writable pidfile 的问题，原因在于 debian 在 wsl 中 root 用户运行 redis 生成的 pidfile 权限是 666，实际上应该为 644。估计是一个 bug，github issue 中有人提过 <https://github.com/microsoft/WSL/issues/4945>。目前暂时的解决方案是在 redis 启动前修改 umask 值，之后在修改回来。

编辑 /etc/init.d/redis-server，将 touch $PIDFILE 修改为

```bash
umask 022
touch $PIDFILE
umask 000 # root 用户默认为000
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
