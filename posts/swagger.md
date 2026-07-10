---
title: swagger
tags:
  - Tool
  - Swagger
categories:
  - tool
date: 2025-11-22 00:00:00
katex: true
---

在后端开发中，接口文档与代码难以同步是常见的问题。Swagger 通过“代码即文档”的方式解决了这一点：开发者编写特定格式的代码注释，工具自动生成标准化的 API 文档和交互式调试页面。

本文主要介绍如何在 Go 后端项目中集成 Swagger。

# 1. 环境准备

在项目根目下执行安装命令：

```sh
# 1. 安装文档生成器 (CLI)
go install github.com/swaggo/swag/cmd/swag@latest

# 2. 安装 Gin 适配器
go get github.com/swaggo/gin-swagger@latest
go get github.com/swaggo/files@latest
```

# 2. swagger注释语法详解

Swagger 的核心在于写对注释。注释分为全局配置和接口配置。

## 2.1 全局配置 (main.go)

放在 `main` 函数上方，用于定义文档的通用信息。

```Go
// @title           项目名称 (如: 支付系统 API)
// @version         1.0
// @description     这里写项目的详细描述...
// @host            localhost:8080
// @BasePath        /api/v1
func main() { //... }
```

## 2.2 接口配置 (Controller)

放在 `Controller/Handler` 函数上方。参考下方的语法拆解。

基础语法结构:

```go
// @Key          Value...
```

常用标签速查表

| 标签         | 必填 | 说明                 | 示例                          |
|--------------|------|----------------------|-------------------------------|
| @Summary     | 是   | 接口简短标题         | 获取用户列表                  |
| @Description | 否   | 详细描述             | 支持分页，支持按姓名搜索      |
| @Tags        | 否   | 接口分组(分类)       | User                          |
| @Accept      | 否   | 请求数据类型         | json, multipart/form-data     |
| @Produce     | 否   | 响应数据类型         | json                          |
| @Param       | 否   | 参数定义             | 见下方详解                    |
| @Success/@failure     | 是   | 成功/失败响应             | 200 {object} User             |
| @Router      | 是   | 路由路径 [方法]            | /users [get]                  |

**重点 1：@Param 参数详解**

格式如下：

```Plaintext
// @Param  参数名  参数位置  参数类型  是否必填  "参数描述"
```

1. 参数位置 (In) 有哪几种？
path: 路径参数，如 /users/{id} 中的 id。
query: URL 问号后面的参数，如 /users?page=1。
body: POST/PUT 请求体，通常对应一个结构体。
header: 请求头参数，如 Authorization。
2. 常见场景写法示例：
场景 A：路径参数 (GET /users/:id)
```go
// @Param  id  path  int  true  "用户ID"
```

场景 B：查询参数 (GET /users?name=xxx)
```go
// @Param  name  query  string  false  "用户名"
```

场景 C：JSON 请求体 (POST /users)注意：当位置是 body 时，参数类型是结构体名。
```go
// @Param  request  body  model.CreateUserRequest  true  "注册信息"
```

**重点 2：@Success 响应详解**

格式如下：

```Plaintext
// @Success  HTTP状态码  {数据类型}  返回结构体  "可选描述"
```

* 返回对象：// @Success 200 {object} model.User
* 返回数组：// @Success 200 {array} model.User
* 返回字符串：// @Success 200 {string} string "ok"
* 出错响应：// @Failure 400 {object} map[string]string

# 3. 完整代码示例

下面展示两个典型接口：一个简单的 GET，一个复杂的 POST。

```go
package main

// LoginReq 登录请求参数
type LoginReq struct {
    Username string `json:"username" example:"admin"` // example 字段会在文档中显示默认值
    Password string `json:"password" example:"123456"`
}

// User 用户响应结构
type User struct {
    ID   int    `json:"id"`
    Name string `json:"name"`
}

// GetUserList 获取用户列表 (Query 参数示例)
// @Summary      获取用户列表
// @Tags         User
// @Param        page      query    int     false  "页码"
// @Param        keyword   query    string  false  "搜索关键字"
// @Success      200       {array}  User
// @Router       /users [get]
func GetUserList(c *gin.Context) { /* ... */ }

// Login 用户登录 (Body 参数示例)
// @Summary      用户登录
// @Tags         Auth
// @Accept       json
// @Produce      json
// @Param        request   body     LoginReq  true  "登录信息"
// @Success      200       {object} User
// @Router       /login [post]
func Login(c *gin.Context) { /* ... */ }
```

> Swagger 基于 函数（handler） 生成文档，而不是基于路由表
> Swagger（swaggo）遵循的规则是：**文档注释必须写在处理该路由的函数（handler）之前。**所以不能使用匿名函数。

# 4. 生成与运行

## 4.1 第一步：生成文档
在 main.go 所在目录执行：

```bash
swag init
```

## 4.2 第二步：引入代码

在 main.go 中加入两行代码：

```go
import (
    // ... 其他包
    // 1. 引入 swag 依赖
    swaggerFiles "github.com/swaggo/files"
    ginSwagger "github.com/swaggo/gin-swagger"

    // 2. 必须引入生成的 docs 包
    _ "your/module/docs"
)

func main() {
    r := gin.Default()
    // ... 路由注册 ...
    
    // 3. 注册 Swagger 路由
    r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
    r.Run(":8080")
}
```

## 4.3 第三步：验证

访问 [swagger 文档](http://localhost:8080/swagger/index.html)。
