---
title: "Windows Terminal 使用 git log / git diff 中文乱码问题"
date: 2020-07-09T16:25:53+08:00
categories: ["乱七八糟"]
tags: ["win10", "Windows Terminal", "git"]
---
## Git 编码设置

```bash
git config --global i18n.commit.encoding utf-8
git config --global i18n.logoutputencoding utf-8
```

## 添加环境变量

设置环境变量 `LESSCHARSET`  为 `utf-8`
