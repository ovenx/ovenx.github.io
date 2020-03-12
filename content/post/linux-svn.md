---
title: Linux 下安装 SVN
date: 2020-03-09 08:00:00
categories: ["程序世界"]
tags: ["linux", "svn"]
---

本文的环境：

* centos 7.6
* svn 1.7.14

### 安装
```bash
yum install subversion
```

### 创建仓库
新建一个名为 test 的代码仓库
```bash
mkdir /var/svn/test
svnadmin create /var/svn/test
```
此时 test 目录的结构如下
```bash
drwxr-xr-x. 6 root root  86 2月  29 13:13 .
drwxrwxrwx. 8 root root 102 3月   9 11:09 ..
drwxr-xr-x. 2 root root  54 3月   9 09:24 conf
drwxr-sr-x. 6 root root 253 2月  29 13:45 db
-r--r--r--. 1 root root   2 2月  29 13:13 format
drwxr-xr-x. 2 root root 231 2月  29 14:01 hooks
drwxr-xr-x. 2 root root  41 2月  29 13:13 locks
-rw-r--r--. 1 root root 229 2月  29 13:13 README.txt
```

### 配置 svn
主要涉及配置文件都在 conf 中
```bash
passwd        # 密码管理
authz         # 权限管理
svnserve.conf # svn服务进程
```

#### 管理用户（passwd)
用户和密码的格式为 `name = password`，每行为一个用户，密码为明文。

添加一个账户名为 test，密码为123456 的用户：
```bash
[users]
test = 123456
```

#### 配置权限 （authz)
```bash
[groups]      # 分组配置
group1 = test
group2 = test1, test2

[/]           # / 表示整个目录
test = rw     # 读写权限
test1 = r     # 只有读权限
test2 = w     # 只有写权限
@group1 = rw  # 分组 group1 内所有用户有读写权限
* =           # 其它用户没有任何权限
```

#### 服务配置（svnserve.conf)
打开下面 4 个注释
```bash
anon-access = read   # 匿名用户无权访问
auth-access = write  # 认证用户可读写
password-db = passwd # 指定用户认证密码文件
authz-db = authz     # 指定权限配置文件
```

### 启动服务
```bash
# -d 服务后台运行 
# -r 指定工作目录
# 注意这里不需要指定到具体哪个仓库 如 /var/svn/test
# 如需指定端口可以使用 --listen-port 3691
svnserve -d -r /var/svn    
```

### 防火墙处理
svn 服务默认的是 `3690` 端口
```bash
firewall-cmd --zone=public --add-port=3690/tcp --permanent # 开启 3690 端口
firewall-cmd --reload                                      # 重启
```
当然也可以直接关闭防火墙，但是不推荐使用
```
systemctl stop firewalld
```

### 客户端连接
客户端可以直接使用 svn://ip:3690/test 来 checkout test 仓库，默认端口可以忽略

### 配置 hook

#### 自动检出
svn 服务端并不是以原文件来存储的，而是会以特殊的格式（FSFS，BDB）进行版本存储。所以如果我们需要直接在服务器中运行代码程序时需要先 checkout 检出代码，还是以test为例：
```bash
# 把 test 仓库的代码检出到 /var/www/test 目录
svn checkout svn://127.0.0.1/test /var/www/test  
```
但是这样每次检出会很麻烦，使用 svn 的 hook 可以实现自动检出。

在 `/var/svn/test/hooks` 目录下新建一个 post-commit 文件，post-commit 文件添加内容为：
```bash
#!/bin/sh
EPOS="$1"           # 仓库
REV="$2"            # 版本号
export.UTF-8        # 编码
SVN=/usr/bin/svn    # svn 
WEB=/var/www/test   # 要更新的项目目录
$SVN update $WEB --username test --password 1234567
```

为 post-commit 添加执行权限

```bash
chmod +x /var/project/test/hooks/post-commit
```

####  commit 限制
默认 svn 提交文件 commit 是没有限制，也就是说可以留空。但通常我们都会限制要求必须填写 commit，以免造成后面的混乱。修改 hook 中的 pre-commit 可以实现 commit 的限制

首先复制一份 pre-commit 文件
```bash
cp pre-commit.tmpl pre-commit
```
修改 pre-commit 文件
```bash
#!/bin/sh
REPOS="$1"
TXN="$2"

SVNLOOK=/usr/bin/svnlook  
LOGMSG=`$SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c`
if [ "$LOGMSG" -lt 10 ];then 
   echo "提交失败： 注释不能低于10个字符" 1>&2 
   exit 1
fi
```

为 pre-commit 添加执行权限

```bash
chmod +x /var/project/test/hooks/pre-commit
```