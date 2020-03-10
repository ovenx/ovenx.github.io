---
title: linux 升级 python3
date: 2020-02-18 10:07:00
categories:
- 程序世界
tags:
- centos
- python3
---
本文的系统版本 `centos7 x64`，默认的 python 版本为 `2.7`，升级之后的版本为 `3.8.1`

### 安装依赖包
```bash
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
yum install libffi-devel # 缺少这个 make 时会报 ModuleNotFound：No module named '_ctypes' 错误
```


### 下载源码包
```bash
wget https://www.python.org/ftp/python/3.8.1/Python-3.8.1.tgz
tar -zxvf Python-3.8.1.tgz
```

### 编译
```bash
cd Python-3.8.1.tgz
./configure
make && make instal
```

### 修改默认 python 及 pip 命令
```bash
mv /usr/bin/python /usr/bin/python.bak
ln -s /usr/local/bin/python3 /usr/bin/python
mv /usr/bin/pip /usr/bin/pip.bak
ln -s /usr/local/bin/pip3 /usr/bin/pip
```

### 处理 yum
由于 yum 需要使用 python 2.7，如果将默认的 python 修改为 python3 版本就会出现问题，这里需要修改两个地方
```bash
vim /usr/libexec/urlgrabber-ext-down
vim /usr/bin/yum
```
把头部的 `/usr/bin/python` 修改为 `/usr/bin/pyhon2.7`