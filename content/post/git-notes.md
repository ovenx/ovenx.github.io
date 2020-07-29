---
title: Git 常用操作笔记
date: 2020-07-29T08:00:00+08:00
categories: ["程序世界"]
tags: ["git"]
---

## 提交 pull request 的最佳实践

### 1. 首先 fork 仓库

添加一个自己的仓库副本

### 2. 添加 upstream source

```bash
git remote add upstrem URL
```

### 3. 创建一个 dev 分支

```bash
git checkout -b dev
git commit -m "add new feature"
```

### 4. 开发及测试完成后 Rebase

```bash
git fetch upstream
git rebase upstream/master
```

### 5. 推送本地开发分支

```bash
git push origin dev
```

### 6. 发起 pull request

在 github 上发起 pull request，选择 dev 分支，填好对应的 description

## Git回滚代码到某个 commit

回退命令：

```bash
git reset --hard HEAD^      #回退到上个版本
git reset --hard HEAD~3     #回退到前 3 次提交之前，以此类推，回退到 n 次提交之前
git reset --hard commit_id  #退到/进到指定 commit
```

回退版本后直接 push 会出错，需要使用 --force 强制 push

```bash
git push origin master --force
```

## clone 某个分支

```bash
git clone -b branch
```

## 合并多个 commit

```bash
git rebase -i commit_id         # 会合并此次提交之后所有的提交为一个提交
git rebase -i commit_1 commit_2 # 会合 commit_1 到 commit_2 之间的记录 不包含 commit_1 包含 commit_2
git rebase -i HEAD~3            # 合并最近的 3 个 commit
git rebase --continue           # 解决冲突后，继续执行 rebase
git rebase --abort              # 终止合并
```

常用的 commit 操作命令

```bash
pick：保留该commit（缩写:p）
reword：保留该commit，但我需要修改该commit的注释（缩写:r）
edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
squash：将该commit和前一个commit合并（缩写:s）
fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
exec：执行shell命令（缩写:x）
drop：我要丢弃该commit（缩写:d）
```

## 取消本地文件 track

```bash
git rm -r --cached .      # 取消所有文件的跟踪，保留文件
git rm -r --f .           # 取消所有文件的跟踪，删除本地文件
git rm --cached test.file # 取消 test.file 的跟踪，保留文件
git rm --f test.file      # 取消 test.file 的跟踪，删除本地文件
```

添加 `test.file` 到 .gitignore 之后提交
