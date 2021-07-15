---
title: "MySQL 设置默认编码为 utf8mb4"
date: 2020-06-30T10:37:16+08:00
categories: ["程序世界"]
tags: ["mysql", "utf8mb4"]
---

## mariadb

修改配置文件

```bash
vim /etc/my.conf.d/server.conf
[mysqld]
character_set_server=utf8mb4
collation-server=utf8mb4_general_ci
init_connect='SET NAMES utf8mb4'
skip-character-set-client-handshake=true
```

重启 mariadb

```bash
systemctl restart mariadb
```

查看编码

```bash
SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
```

## mysql

修改配置文件

```bash
vim /etc/my.cnf

[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
[mysqld]
character_set_server=utf8mb4
collation-server=utf8mb4_general_ci
init_connect='SET NAMES utf8mb4'
skip-character-set-client-handshake=true
```

重启 mariadb

```bash
systemctl restart mysqld
```

查看编码

```bash
SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
```
