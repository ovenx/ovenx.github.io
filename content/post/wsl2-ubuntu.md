---
title: WSL2 Ubuntu 配置
date: 2020-06-23T18:24:00+08:00
categories: ["程序世界"]
tags: ["wsl", "php", "debian"]
---

系统版本：Ubuntu-20.04

## 配置国内源

修改 source.list

```bash
cp /etc/apt/source.list /etc/apt/source.list.bak
vi source.list

# 复制下面内容到 source.list
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

更新源

```bash
apt update
apt upgrade
```

## python3

```bash
sudo apt install python-is-python3
```

## nginx

```bash
apt-get install nginx
service nginx start
```

## php

```bash
apt update
apt install php php-cli php-fpm php-json php-common php-mysql php-zip php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-dev
service php7.4-fpm start
```

## redis

安装 redis

```bash
apt-get install redis-server
apt-get install php-redis
```

设置密码

```bash
vi /etc/redis/redis.conf
# uncomment requirepass 123456
```

启动 redis

```bash
service redis-server start
```

## swoole

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

## docker

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo service docker start
```

## node

```bash
# https://github.com/nodesource/distributions
curl -sL https://deb.nodesource.com/setup_current.x | sudo -E bash -
sudo apt-get install -y nodejs
```

## zsh

```bash
apt-get install zsh
```

替换默认 shell

```bash
chsh -s /bin/zsh
```

安装 oh my zsh

```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

使用主题

```bash
vi ~/.zshrc
ZSH_THEME="agnoster"
cd ~
```
