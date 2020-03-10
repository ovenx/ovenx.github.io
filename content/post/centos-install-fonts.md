---
title: Centos 安装字体
date: 2020-02-17 23:05:00
categories: ["程序世界"]
tags: ["centos", "fonts"]
---
本文的系统版本 `centos7 x64`

### 查看字体
```bash
fc-list
```

### 安装字体
```bash
cp xxx.ttf /usr/share/fonts/ # copy fonts to /usr/share/fonts
```

### 建立字体缓存
```bash
cd /usr/share/fonts/
chmod 755 *.ttf # 修改字体权限
mkfontscale # 依赖 mkfonts
mkfontdir
fc-cache -fv # 依赖 fontconfig
```
