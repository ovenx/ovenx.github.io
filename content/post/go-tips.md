---
title: "Go Tips"
date: 2021-07-19T17:00:57+08:00
categories: ["程序世界"]
tags: ["golang"]
---

## gomod

### 开启 gomod

1.13 以前

```bash
GO111MODULE=on
GOPROXY=https://goproxy.io
```

1.13 以后

```bash
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```

### go get 使用

```bash
go get -u # 升级包版本
go get -u=path # 升级到最新的修订版
go get package@version # 升级到某个版本
```

### go mod 基本操作

```bash
go mod init mod_name  ## 初始化 mod
go mod download  ## 下载 mod 到本地 cache $GOPATH/pkg/mod 和 $GOPATH/pkg/sum下
go mod edit # 编辑 go.mod 文件
go mod tidy # 删除不使用的 mod
go mod vendor # 生成 vendor 目录
go mod verify # 验证依赖
go mod why # 查找依赖
go clean --modcache # 清楚 module 缓存
```

## 格式化整个项目代码

```bash
gofmt -s -w -l .
```

## build 不同平台

Mac 下编译 Linux, Windows 平台的 64 位可执行程序：

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o output_name main.go 
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build` main.go
```

Linux 下编译 Mac, Windows平台的 64 位可执行程序：

```bash
CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 go build main.go
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build` main.go
```

Windows 下编译 Mac, Linux 平台的 64 位可执行程序：

```bash
SET CGO_ENABLED=0 SET GOOS=darwin3 GOARCH=amd64 go build main.go 
SET CGO_ENABLED=0 SET GOOS=linux GOARCH=amd64 go build main.go
```
