---
title: "PHP 配合 Rsync 同步推送代码到服务器"
date: 2020-03-12T10:00:41+08:00
categories: ["程序世界"]
tags: ["linux", "rsync", "php"]
draft: false
---

## 前言

之前搭建公司开发环境有一个需求，需要将本地测试服务器中的代码同步推送到线上的服务器。后面开始研究 rsync，中途踩了很多坑，故在此记录下一些主要的操作和注意事项

## Rsync 设置


### 本地同步

同步单个文件
```bash
rsync -avz /etc/passwd /tmp/    # 同步 passwd 文件到 tmp 目录下
```

同步目录下所有文件（`包含目录本身`）
```bash
mkdir /root/test1
echo "test1" > /root/test1/file1
rsync -avz /root/test1 /tmp/    # 同步 test1 目录 文件到 tmp 目录下
```

同步目录下所有文件（`不包含目录本身`）
```bash
mkdir /root/test2
echo "test" > /root/test2/file2
rsync -avz /root/test2/ /tmp/    # 同步 test2 目录 文件到 tmp 目录下
```
> 特别注意后面以 `/` 结尾不包含目录，没有 `/` 则包含目录

### 通过 SSH 同步到远程服务器
```bash
# 本地服务器
echo "test" > /root/rsync_file 
rsync -avz  /root/rsync_file root@remote_ip:/root/rsync_file  # 推送到远程服务器

# 也可以指定 key 文件实现免密处理
rsync -avz -e "ssh /rsync/ssh-key" /root/rsync_file  root@remote_ip:/root/rsync_file

# 远程服务器
cat /root/rsync_file
test
```

> 使用 ssh 可以实现推送到远程服务器的工作，但是存在很多问题。最主要的一点是 PHP 脚本执行 ssh 时会存在权限的问题。

### 使用 Rsync 进程方式传输

这是最常用的使用方式，需要服务端配置运行相应的 rsync 进程

**服务端**

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
path = /var/www/test
```

创建虚拟用户
```bash
useradd -M -s /sbin/nologin rsync
```

创建备份目录
```bash
mkdir -p /var/www/test/
chown -R rsync.rsync /var/www/test/ 
```

创建虚拟用户密码文件
```bash
# 用户：rsync_backup 密码：123456
echo "rsync_backup:123456" >/etc/rsync.password    
chmod 600 /etc/rsync.password
```

启动服务
```bash
systemctl start rsyncd    # 启动服务
systemctl enable rsyncd   # 开机启动
netstat -lntp             # 检查对应的端口
```

防火墙设置
```bash
firewall-cmd --zone=public --add-port=873/tcp --permanent  # 开启 873 端口
firewall-cmd --reload                                      # 重启
```
> 注意如果服务器是阿里云一定要记得在阿里云的安全策略中加入 873 端口


**客户端**

设置密码文件
```bash
echo "123456" >/etc/rsync.password   # 这里的密码要和服务端保持一致
chmod 600 /etc/rsync.password
```

开始推送

```bash
# ::test 指的是上面 /etc/rsyncd.conf 中的模块名 
# --delete 删除那些 DST 中 SRC 没有的文件
rsync -avz --delete --password-file=/etc/rsync.password /var/www/test/ rsync_backup@remote_ip::test
```

使用 --exclude-from 排除不需要同步的目录或文件

```bash
mkdir /etc/rsync
vi exclude.list
```

加入需要排除的文件或文件夹

```bash
runtime/*
config/database.php
```

最终的命令
```bash
rsync -avz --delete --exclude-from=/etc/rsync/exclude.list --password-file=/etc/rsync.password /var/www/test/ rsync_backup@remote_ip::test
```


## PHP 调用 Rsync 服务

PHP 配合 Rsync 可以做到指定文件的同步。构建一个同步推送页面，开发人员输入需要同步的文件或者目录，PHP 后端进行批量同步处理。这样做的好处是不需要每次都通过 Linux 命令行来推送代码，多人协作时也更加方便。具体的逻辑我这里就不再描述，核心就是使用用 PHP 的 `exec` 方法执行 Liunx 命令
```php
 exec('rsync -avz --delete --exclude-from=/etc/rsync/exclude.list --password-file=/etc/rsnyc.password rsync_backup@remote_ip::test 2>&1', $output, $return_var);
```


