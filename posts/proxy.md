---
title: proxy
tags:
  - DesignPattern
  - ProxyPattern
categories:
  - Design Pattern
date: 2025-10-31 00:00:00
katex: true
---

# 1. 什么是代理模式

代理模式是一种结构型设计模式，它通过引入一个代理对象来控制对另一个对象的访问。代理对象可以在客户端和目标对象之间添加额外的功能，如延迟加载、访问控制、日志记录等。

# 2. 为什么需要代理模式

代理模式的核心价值在于**在不改变原始对象（目标对象）代码的前提下，增加额外的控制和功能**。它像一个中介，隔离了客户端和真实对象，并在中间“做手脚”。


# 3. 代理模式的实现 (Go)

我们用 Go 实现一个**保护代理**的例子。假设我们有一个 Web 服务器，希望通过一个 Nginx 代理来限制用户的访问速率。

1. 客户端、代理和真实服务都应遵循同一个接口，这样代理才能“伪装”成真实服务。

```go
// Server 是代理和真实服务器的通用接口
type Server interface {
	HandleRequest(url, method string) (int, string)
}
```

2. 创建真实服务 (RealSubject)

这是我们真正要保护的后端应用。

```go
// Application 是我们的后端真实服务
type Application struct{}

func (a *Application) HandleRequest(url, method string) (int, string) {
	if url == "/app/status" && method == "GET" {
		return 200, "Ok"
	}
	if url == "/create/user" && method == "POST" {
		return 201, "User Created"
	}
	return 404, "Not Found"
}
```

3. 创建代理 (Proxy)

Nginx 代理持有对真实服务的引用，并在处理请求前增加了速率限制的逻辑。

```go
import "fmt"

// Nginx 是一个代理，它控制对 Application 的访问
type Nginx struct {
	application      *Application
	maxAllowedReq    int
	rateLimiter      map[string]int
}

func NewNginxServer() *Nginx {
	return &Nginx{
		application:      &Application{},
		maxAllowedReq:    2,
		rateLimiter:      make(map[string]int),
	}
}

// checkRateLimit 检查并更新请求速率
func (n *Nginx) checkRateLimit(url string) bool {
	n.rateLimiter[url]++
	if n.rateLimiter[url] > n.maxAllowedReq {
		return false // 超出速率限制
	}
	return true
}

// HandleRequest 代理的核心方法
func (n *Nginx) HandleRequest(url, method string) (int, string) {
	// 在转发请求前，执行额外的逻辑（访问控制）
	if !n.checkRateLimit(url) {
		return 429, "Too Many Requests"
	}

	// 将请求转发给真实服务
	return n.application.HandleRequest(url, method)
}
```

4. 客户端调用

客户端只与代理（Nginx）交互，并不知道真实服务（Application）的存在。

```go
func main() {
	nginx := NewNginxServer()

	// 第一次请求 /app/status
	code, body := nginx.HandleRequest("/app/status", "GET")
	fmt.Printf("Url: /app/status, Code: %d, Body: %s\n", code, body)

	// 第二次请求 /app/status
	code, body = nginx.HandleRequest("/app/status", "GET")
	fmt.Printf("Url: /app/status, Code: %d, Body: %s\n", code, body)

	// 第三次请求 /app/status，将被限流
	code, body = nginx.HandleRequest("/app/status", "GET")
	fmt.Printf("Url: /app/status, Code: %d, Body: %s\n", code, body)
}
```

类图

{% mermaid %}
classDiagram
    class Client
    class Server {
        <<interface>>
        +HandleRequest(url, method)
    }
    class Application {
        +HandleRequest(url, method)
    }
    class Nginx {
        -application: *Application
        -maxAllowedReq: int
        -rateLimiter: map
        +HandleRequest(url, method)
        -checkRateLimit(url): bool
    }

    Client --> Server : uses
    Server <|.. Application : implements
    Server <|.. Nginx : implements
    Nginx o-- Application : aggregates
{% endmermaid %}


