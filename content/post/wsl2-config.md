---
title: WSL2 常见问题
date: 2020-12-03T18:24:00+08:00
categories: ["程序世界"]
tags: ["wsl"]
---

## 安装 wsl2

开启 wsl2

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

设置默认 wsl2

```bash
wsl --set-default-version 2
```

设置默认发行版

```bash
wsl -s Ubuntu-20.04
```

## 启动和停止 wsl 服务

```bash
wsl --shutdown
# net stop LxssManager
```

## 文件权限问题

由于 windows 文件系统是 NTFS, 在 wsl 在所有的文件权限都会是 777。解决方法是修改配置文件 /etc/wsl.conf

```bash
[automount]
enabled = true
options = "metadata,umask=22,fmask=11"
mountFsTab = false
```

重启后文件的权限就不再都是 777 了。

## wsl 文件在 windows 系统中的位置

可以把它映射到网络位置

```bash
\\wsl$
```

## 设置默认 root 登录

```bash
debian config --default-user root
```

## 修改 wsl 位置

关闭 wsl

```bash
wsl --shutdown
```

查看系统

```bash
wsl -l -v
```

导出系统

```bash
wsl --export Debian D:\ubuntu.tar
```

删除原有系统

```bash
wsl --unregister Ubuntu-20.04
```

导入新新系统

```bash
wsl --import Ubuntu-20.04 D:\ubuntu D:\ubuntu.tar
wsl -l -v
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
