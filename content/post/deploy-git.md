---
title: Linux 下搭建 git 服务器
date: 2020-03-16T09:00:00+08:00
categories: ["程序世界"]
tags: ["git"]
---

## 安装 git

```bash
yum install git git-core
```

## 创建 git 用户

为了服务器的安全，一般不会直接使用 root 用户来运行 git 服务。我们可以新建一个 git 用户专门用来执行 git 服务。

```shell
# 添加一个 git 用户
sudo adduser git

# 设置密码
passwd git

# 禁用shell登录
sudo vi /etc/passwd
git: x:1001:1002:,,,:/home/git:/bin/bash # 修改为下面的
git: x:1001:1002:,,,:/home/git:/usr/bin/git-shell
```

这样，git 用户可以正常通过 ssh 使用 git，但无法登录 shell，因为我们为 git 用户指定的 git-shell每次一登录就自动退出。

## 创建证书登录

收集所有需要登录的用户的公钥，就是他们自己的id_rsa.pub文件，把所有公钥导入到` /home/git/.ssh/authorized_keys` 文件里，一行一个。
> 注意 `authorized_keys` 的权限要设置为 `600`，`.ssh` 文件夹设置为 `700`,没有这个文件的话可以自己创建

## 新建 git 仓库

```bash
mkdir /opt/git
cd /opt/git
git init --bare home.git

# 文件夹权限
chown -R git:git home.git
cd /home.git/hooks
sudo vi post-receive
#写入以下内容
#!/bin/sh
GIT_WORK_TREE=/var/www/home  git checkout -f

# 写入权限
chmod +x post-receive

# 建立web目录，如果目录不存在，git不会创建目录的
mkdir /var/www/home  -p

# web目录的文件夹权限
chown -R git:git /var/www/home
```

## 克隆远程仓库

```bash
git clone git@server:/opt/home.git
```

## Reference

- [搭建 Git 服务器](https://www.liaoxuefeng.com/wiki/896043488029600/899998870925664)
- [使用 Git 来部署一个 Web 站点笔记](https://rmingwang.com/using-git-to-deploy-a-web-site.html)
