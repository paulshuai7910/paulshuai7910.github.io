---
title: MongoDB 工具文档
date: 2025-04-09 14:40:27
tags:
---

## 主要工具

- **bsondump** - _以人类可读格式显示 BSON 文件_
- **mongoimport** - _将 JSON、TSV 或 CSV 数据转换并插入到集合中_
- **mongoexport** - _将现有集合写入 CSV 或 JSON 格式_
- **mongodump/mongorestore** - _将 MongoDB 备份以 .BSON 格式转储到磁盘，或将它们恢复到活动数据库_
- **mongostat** - _监控活动的 MongoDB 服务器、副本集或分片集群_
- **mongofiles** - _在 [GridFS](http://docs.mongodb.org/manual/core/gridfs/) 中读取、写入、删除或更新文件_
- **mongotop** - _监控 MongoDB 服务器上的读写活动_

如有任何错误、改进或新功能请求，请报告至 https://jira.mongodb.org/browse/TOOLS

## 构建工具

我们目前使用 Go 1.15 版本构建工具。其他 Go 版本可能可用但未经测试。

直接使用 `go get` 构建工具将不起作用。要构建它们，建议首先克隆此仓库：

```
git clone https://github.com/mongodb/mongo-tools
cd mongo-tools
```

然后运行 `./make build` 来构建所有工具，将它们放置在仓库内的 `bin` 目录中。运行 `./bin/mongodump --help` 来验证构建的二进制文件是否正常工作。

您也可以使用 `-pkgs` 选项构建部分工具。例如，`./make build -pkgs=mongodump,mongorestore` 仅构建 `mongodump` 和 `mongorestore`。

要使用此仓库中的构建/测试脚本，您**_必须_**将 GOROOT 设置为您的 Go 根目录。这可能取决于您安装 Go 的方式。

```
export GOROOT=/usr/local/go
```

### 仅限 Mac OS：

运行构建的二进制文件时，如果进程立即被终止，并且您看到以下输出：  
`zsh: killed ./bin/mongodump --help`  
那么您需要对二进制文件进行签名才能运行它：  
`codesign --force --sign - bin/mongodump`

您还可以选择配置终端应用程序的安全策略，这样就不需要签名：  
(在 macOS Sonoma 中) 系统设置 > 隐私与安全 > 开发者工具

## 更新依赖

从版本 100.3.1 开始，工具使用 `go mod` 管理依赖。所有依赖都列在 `go.mod` 文件中，并直接存放在 `vendor` 目录中。

要对依赖进行更改，您首先需要修改 `go.mod` 文件。您可以手动编辑该文件以添加/更新/删除条目，或者在仓库目录中运行以下命令：

```
go mod edit -require=<package>@<version>  # 用于添加或更新依赖
go mod edit -droprequire=<package>        # 用于删除依赖
```

然后运行 `go mod vendor -v` 来重建 `vendor` 目录以匹配更改后的 `go.mod` 文件。

可选地，运行 `go mod tidy -v` 以确保 `go.mod` 文件与 `mongo-tools` 源代码匹配。

## 贡献

请参阅我们的[贡献者指南](CONTRIBUTING.md)。

## 文档

请参阅 MongoDB 包的[文档](https://docs.mongodb.org/database-tools/)。

对于旧版本 MongoDB 的文档，请参考该版本的 [MongoDB 服务器手册](docs.mongodb.com/manual)：

- [MongoDB 4.2 工具](https://docs.mongodb.org/v4.2/reference/program)
- [MongoDB 4.0 工具](https://docs.mongodb.org/v4.0/reference/program)
- [MongoDB 3.6 工具](https://docs.mongodb.org/v3.6/reference/program)

## 添加新平台支持

请参阅我们的[添加新平台支持指南](PLATFORMSUPPORT.md)。

## 将更改引入服务器仓库

请参阅我们的[将更改引入服务器仓库](SERVERVENDORING.md)。
