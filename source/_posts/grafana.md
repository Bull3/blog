---
title: 从源码构建Grafana (只针对windows系统)
categories: 全栈类
description: 描述
tags:
  - 前端
  - 后端
date: 2019-12-04 14:46:37
---

## 1. grafana源代码及文档

> 从[grafana发布包](https://github.com/grafana/grafana/releases)中选择一个版本, 下载压缩包后解压缩
> 开发文档可参考[grafana官方文档](https://grafana.com/docs/)
> [旧版构建指南](https://grafana.com/docs/grafana/v4.5/project/building_from_source/) 
> [新版构建指南](https://github.com/grafana/grafana/blob/master/contribute/developer-guide.md)

## 2. 构建后端

### 2.1 安装依赖

> 项目必需的依赖项
>* [Git](https://git-scm.com/)
>* [Go](https://gomirrors.org/)（请参阅[go.mod](https://github.com/grafana/grafana/blob/master/go.mod#L3)以获取最低要求的版本）
>* [Node.js](https://nodejs.org/en/)
>* [Yarn](https://yarnpkg.com/lang/en/)

> 构建后端所需的依赖项
>* Grafana后端包括需要GCC编译的Sqlite3。建议安装[TDM-GCC](http://tdm-gcc.tdragon.net/download)。
>* node-gyp是Node.js本机插件构建工具，它需要额外的依赖项才能在Windows上安装。以管理员身份运行powershell，输入：npm --add-python-to-path='true' --debug install --global windows-build-tools

### 2.2 编译后端

由于golang网站在国内被墙, go的一些依赖无法安装, 需要设置go的全局代理,
新建2个windows环境变量, 设置 GO111MODULE 为 on , 设置 GOPROXY 为 https://goproxy.io (七牛也有国内代理 https://goproxy.cn)

```dos
go run build.go setup
go run build.go build    # (或者 'go build ./pkg/cmd/grafana-server')
# 编译完成会在bin\windows-amd64目录下的生成grafana-server.exe等文件
```

### 2.3 实时编译后端

```bat
go get github.com/Unknwon/bra
bra run
```

## 3. 构建前端

```bat
yarn # 安装package.json中的依赖
yarn start # 编译并打包前端代码
```

## 4. 启动服务

### 4.1 开发配置

在conf目录中创建custom.ini，以覆盖默认配置选项。您只需要添加要覆盖的选项。配置文件按以下顺序应用：
```bat
1. grafana.ini
2. custom.ini
```

### 4.2 启动服务

在项目根目录下运行.\bin\windows-amd64\grafana-server
默认访问http://localhost:3000/ 即可看到页面, 用户名密码均为admin, 端口号可在上面的ini配置中修改