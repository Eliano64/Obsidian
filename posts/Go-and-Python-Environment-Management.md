---
title: Go and Python Environment Management
tags:
  - Environment
categories:
  - Tool
date: 2026-06-24 00:00:00
katex: true
---

# Go and Python Environment Management

## 1. Go 常用命令

| 命令 | 用途 |
|---|---|
| `go run .` | 编译并运行当前模块 |
| `go build ./...` | 编译所有包 |
| `go test ./...` | 运行测试 |
| `go mod init <module>` | 初始化模块 |
| `go mod tidy` | 清理并同步依赖 |
| `go get <pkg>` | 添加或更新依赖 |
| `go fmt ./...` | 格式化 |
| `go vet ./...` | 静态检查 |
| `go doc <pkg>` | 查看文档 |

推荐提交前组合：

```bash
go mod tidy
go fmt ./...
go test ./...
```

## 2. Python 环境迁移

导出依赖：

```bash
pip freeze > requirements.txt
```

新环境安装：

```bash
python -m venv .venv
# Linux/macOS
source .venv/bin/activate
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Conda：

```bash
conda env export > environment.yml
conda env create -f environment.yml
```
