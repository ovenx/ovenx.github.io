---
title: 常用工具国内源配置
date: 2020-03-11T09:54:00+08:00
categories: ["程序世界"]
tags: ["source", "linux"]
---

由于国内的网络原因，很多工具（如 `npm`, `pip`）等在国内连接很慢，这里整理了下常用的工具国内源

## Linux

### CentOS
相关地址：[http://mirrors.aliyun.com/repo/](http://mirrors.aliyun.com/repo/)

首先备份
```bash
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

下载源
```bash
# http://mirrors.aliyun.com/repo/Centos-6.repo
# http://mirrors.aliyun.com/repo/Centos-7.repo
# http://mirrors.aliyun.com/repo/Centos-8.repo
wget http://mirrors.aliyun.com/repo/Centos-7.repo
mv Centos-7.repo /etc/yum.repos.d/CentOS-Base.repo
```

生成缓存
```bash
yum clean all   # 清除系统所有的 yum 缓存
yum makecache   # 生成 yum 缓存
yum update      # 更新 yum 
```


## PIP

相关地址：[https://developer.aliyun.com/mirror/pypi](https://developer.aliyun.com/mirror/pypi)

修改配置文件 `~/.pip/pip.conf`
```bash
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com
```

## Composer

相关地址：[https://developer.aliyun.com/composer](https://developer.aliyun.com/composer)

```bash
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

## NPM 

相关地址：[https://developer.aliyun.com/mirror/NPM?from=tnpm](https://developer.aliyun.com/mirror/NPM?from=tnpm)

```bash
npm config set registry https://registry.npm.taobao.org
```

## GEM

相关地址： [https://gems.ruby-china.com](https://gems.ruby-china.com)

```bash
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```

