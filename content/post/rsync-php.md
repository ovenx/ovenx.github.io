---
title: "PHP 配合 Rsync 同步推送代码到服务器"
date: 2020-03-11T13:59:41+08:00
categories: ["程序世界"]
tags: ["linux", "rsync", "php"]
draft: true
---

## 前言

之前搭建公司开发环境有一个需求，需要将本地测试服务器中的代码同步推送到线上的服务器。后面开始研究 rsync，中途踩了很多坑，故在此记录下一些主要的操作和注意事项

## Rsync 设置


### 本地同步

* 同步单个文件
```bash
rsync -avz /etc/passwd /tmp/    # 同步 passwd 文件到 tmp 目录下
```

* 同步目录下所有文件（`包含目录本身`）
```bash
mkdir /root/test1
echo "test1" > /root/test1/file1
rsync -avz /root/test1 /tmp/    # 同步 test1 目录 文件到 tmp 目录下
```

* 同步目录下所有文件（`不包含目录本身`）
```bash
mkdir /root/test2
echo "test" > /root/test/file2
rsync -avz /root/test2/ /tmp/    # 同步 test2 目录 文件到 tmp 目录下
```

### 通过 ssh 同步到远程服务器
```bash
# 本地服务器
echo "test" > /root/rsync_file 
rsync -avz  /root/rsync_file root@remote_server_ip:/root/rsync_file  # 推送到远程服务器

# 远程服务器
cat /root/rsync_file
test
```

> 使用 ssh 可以实现推送到远程服务器的工作，但是存在很多问题。最主要的一点是 PHP 脚本执行 ssh 时会存在权限的问题。

### 使用 rsync 进程方式传输
这是最常用的使用方式，需要服务端配置运行相应的 rsync 进程

#### 服务端配置
首先服务端需要安装 rsync 
```bash
yum install rsync -y
```
编辑配置文件 `/etc/rsyncd.cnf`  
```bash
# 全局配置
uid = rsync
gid = rsync
port = 873
fake super = yes
use chroot = no
max connections = 200
timeout = 600
ignore errors
read only = false
list = false
auth users = rsync_backup
secrets file = /etc/rsync.password
log file = /var/log/rsyncd.log

# 模块配置
[test]
comment = welcome to  rsync!
path = /var/www/test/
```

创建虚拟用户
```bash
useradd -M -s /sbin/nologin rsync
```

创建备份目录
```bash
mkdir /backup
chown -R rsync.rsync /backup/ 
```

创建虚拟用户密码文件
```bash
echo "rsync_backup:123456" >/etc/rsync.password    密码设置为123456
chmod 600 /etc/rsync.password
```

启动服务
```bash
systemctl start rsyncd    # 启动服务
systemctl enable rsyncd   # 开机启动
netstat -lntp             # 检查对应的端口
```


#### 客户端配置


## PHP 调用 Rsync 服务


