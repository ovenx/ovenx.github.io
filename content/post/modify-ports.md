---
title: "修改常用端口"
date: 2021-07-16T10:00:00+08:00
categories: ["程序世界"]
tags: ["port", "ssh", "mysql"]
---

## 修改 ssh 22 端口
```bash
vi /etc/ssh/sshd_config
Port xxxx
sysemctl restart sshd
```

修改 git 对应端口 vi /.git/config
```bash
url = ssh://git@server:xxxx/root/test.git
```

rsync 指定端口
```bash
rsync -e 'ssh -p xxxx'
````

ssh 指定端口
```bash
ssh -p xxxx root@server "systemctl restart heepark"
```

## 修改 MySQL 3306 端口
```bash
vi /etc/my.cnf.d/server.cnf

[mysqld]  
port=xxxx

systemctl restart mariadb
```
