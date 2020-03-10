---
title: hexo+github 搭建个人博客
date: 2019-10-12 08:00:00
categories:
- IT
tags:
- hexo
- github
---

前段时间把博客搬家到了 github 上，这里记录下过程。

### 安装 hexo
```
npm install hexo
hexo init blog
cd blog
npm install
```

常用的命令
```bash
hexo new [layout] <title> # 新建文章
hexo g # 生成静态文件
hexo s  # 启动服务器
hexo d # 部署网站
hexo d -g # 部署之前预先生成静态文件
```

### github 配置

1. github 中创建名为 `用户名.github.io` 的仓库，例如：ovenx.github.io

2. 配置SSH密钥
```bash
ssh-keygen -t rsa -C "your_email@example.com"
```
把生成后好的 id_ras.pub 文件中的内容复制到 github 中的 SSH keys 设置中

3. 设置本地 git 的用户信息
```bash
$ git config --global user.name "username"  #用户名
$ git config --global user.email  "your_email@example.com" #邮箱
```

### hexo 发布到 github 
编辑博客配置文件`_config.yml`

```
deploy:
  type: git
  repo: git@github.com:ovenx/ovenx.github.io.git
  branch: master
```
生成静态文件，发布到 github 仓库
```
hexo d -g # 部署之前预先生成静态文件
```

### 域名配置
1. 创建 CNAME 文件
博客 `source` 目录中新建 `CNAME` 文件，里面填入个人的域名如 blog.xiongwentao.me

2. 配置 DNS
添加一条 CNAME 记录， 值为 blog 仓库的地址，以 cloudflare 为例：
```markdown
| Type  | Name | Content         | TTL  | Proxy status |
| ----- | ---- | --------------- | ---- | ------------ |
| CNAME | blog | ovenx.github.io | Auto | Proxied      |
```

### hexo 多端同步
基本的思路就是把博客编译前的内容也同步 github 中，这样随时随地都可以 clone 下来，快速搭建博客

首页我们需要添加 .gitignore 文件，过滤掉一些不需要的内容
```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

新建一个 hexo 分支
```bash
git init
git branch hexo  
git checkout hexo  
```
添加仓库，推送
```bash
git remote add origin git@github.com:ovenx/ovenx.github.io.git 
git push origin hexo  
```
