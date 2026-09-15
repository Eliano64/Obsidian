---
title: Go Environment Management
tags:
  - Go
categories:
  - Tool
date: 2026-09-15 18:20:00
katex: true
---
# 路径与开关

全部配置项都可以用 `go env` 查看，用 `go env -w 变量=值` 修改。

| 变量              | 作用                             | 默认值或常见取值                                                              |
| --------------- | ------------------------------ | --------------------------------------------------------------------- |
| `GOPATH`        | 工作区根目录，其下的 `bin` 是命令默认安装位置     | `%USERPROFILE%\go`（Windows）、`$HOME/go`                                |
| `GOBIN`         | 命令安装目录，设置后覆盖 `GOPATH/bin`      | 默认为空                                                                  |
| `GOMODCACHE`    | 下载的依赖模块缓存目录                    | `GOPATH\pkg\mod`                                                      |
| `GOPROXY`       | 模块代理，多个值用逗号分隔，`direct` 表示直连源仓库 | 默认 `https://proxy.golang.org,direct`；国内常用 `https://goproxy.cn,direct` |
| `GOSUMDB`       | 模块校验和数据库，用于验证下载内容是否被篡改         | `sum.golang.org`                                                      |
| `GOTOOLCHAIN`   | 是否允许使用 `go.mod` 要求的其它版本工具链     | `auto`                                                                |
| `GOOS`、`GOARCH` | 目标操作系统与架构                      | 随机器而定，如 `linux`、`amd64`                                               |
| `GOENV`         | `go env -w` 写入的文件位置            | `%APPDATA%\go\env`（Windows）、`$HOME/.config/go/env`                    |

`go env -w` 把设置写进上面那个文件，对所有终端生效，不必改系统环境变量；`go env -u 变量` 用于撤销某一项。

`go install pkg@version` 装出的可执行文件放在 `GOBIN`，默认为 `GOPATH/bin`。该目录不在 `PATH` 中时，就会出现"装过却找不到命令"的现象。

# 常用命令

| 命令 | 用途 |
|---|---|
| `go mod init <module>` | 初始化模块，生成 `go.mod` |
| `go mod tidy` | 补齐缺失依赖、删除未使用依赖，并更新 `go.sum` |
| `go mod download` | 只下载依赖、不编译，适合在 CI 中单独缓存 |
| `go mod vendor` | 把依赖复制进 `vendor/`，此后构建优先使用该目录 |
| `go list -m all` | 列出当前模块的全部依赖 |
| `go get <pkg>` | 添加或升级依赖，结果写入 `go.mod` |
| `go run .` | 编译并运行当前包 |
| `go build ./...` | 编译全部包，不安装 |
| `go vet ./...` | 静态检查 |
| `go test ./...` | 运行测试 |
| `go fmt ./...` | 按官方格式排版 |
| `go doc <pkg>` | 查看包文档 |
| `go version -m <binary>` | 查看二进制中记录的模块路径与 Go 版本 |
| `go clean -modcache` | 清空模块缓存，用于缓存损坏或回收磁盘空间 |

# 模块工作流

1. 新项目先执行 `go mod init example.com/demo`，生成的 `go.mod` 写明模块路径与所需的 Go 版本。
2. 代码中 `import` 第三方包后执行 `go mod tidy`。它同时完成三件事：把缺少的依赖写进 `go.mod`、删除不再使用的依赖、更新 `go.sum` 中的校验和。
3. `go.mod` 与 `go.sum` 都要提交。前者记录依赖版本，后者记录校验和，缺任何一个都会让不同机器上的构建结果不一致。
4. 提交前的固定组合：

```bash
go mod tidy
go fmt ./...
go vet ./...
go test ./...
```

在模块模式下，`go get` 只修改 `go.mod` 并把模块下载到缓存，不再像 GOPATH 时代那样把源码放进 `src` 目录。

# 交叉编译

目标平台由 `GOOS` 与 `GOARCH` 决定：

```bash
# Linux / macOS bash
GOOS=linux GOARCH=amd64 go build -o app-linux

# Windows PowerShell
$env:GOOS = 'linux'
$env:GOARCH = 'amd64'
go build -o app-linux
```

再加上 `CGO_ENABLED=0` 会关闭 cgo，得到的二进制不依赖目标机的 libc，适合放进精简基础镜像。