---
title: "PostgreSQL 使用教程"
date: 2021-01-25T17:45:00+08:00
categories: ["程序世界"]
tags: ["postgresql"]
---

## 安装环境
* Linux 版本： centos 7.6
* PostgreSQL 版本：13

## 安装 postgresql

```bash
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86\_64/pgdg-redhat-repo-latest.noarch.rpm  
sudo yum install -y postgresql13-server  
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb  
sudo systemctl enable postgresql-13  
sudo systemctl start postgresql-13
```
[参考来源：https://www.postgresql.org/download/linux/redhat/](https://www.postgresql.org/download/linux/redhat/)

## 添加用户和数据库

### 使用 psql 

```bash
sudo -u postgres psql                              # 切换为 postgres 用户登录 psql
\password                                          # 设置默认密码
create user myuser with password 'mypass';         # 添加用户
create database mydb OWNER myuser;                 # 添加数据库
grant all privileges on database mydb to myuser;   # 添加权限
\q  # quit
```

### 使用 shell 
```bash
sudo -u postgres createuser --superuser myuser;    # 添加用户
sudo -u postgres psql                              # 登录 psql 设置密码
\password myuser
\q
sudo -u postgres createdb -O myuser mydb           # 添加数据库
```

## 登录数据库

### 本地登录

```bash
psql -h 127.0.0.1 -p 5432 mydb myuser
```

### 远程登录

修改 `pg_hba.conf` 配置文件， `vi /var/lib/pgsql/13/data/pg_hba.conf `

找到  `# IPv4 local connections:`，下面添加一行

```bash
host    all             all             0.0.0.0/0               scram-sha-256
```

修改 `postgresql.conf` 配置文件，`vi /var/lib/pgsql/13/data/postgresql.conf`

找到 `#listen_addresses = 'localhost'`，打开注释，修改为

```bash
listen_addresses = '*'
```

最后重启服务即可（** 注意：如果是部署在云服务器上，需要在安全组策略中放行 5432 端口 **）

```bash
systemctl restart postgresql-13
```



## 常用命令
```
ALTER USER myuser WITH PASSWORD 'mypass';  #修改密码
```
