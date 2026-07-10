---
title: chain of responsibility
tags:
  - DesignPattern
  - ChainOfResponsibilityPattern
  - InterfaceOrientedProgramming
categories:
  - Design Pattern
date: 2025-11-08 00:00:00
katex: true
---

# 1. 什么是责任链模式？

责任链模式（Chain of Responsibility Pattern）是一种行为型设计模式，它将请求的发送者和接收者解耦，将这些处理者连成一条链。链上的每个处理者都有一个成员变量来保存对于下一处理者的引用。除了处理请求外，处理者还负责沿着链传递请求。请求会在链上移动，直至所有处理者都有机会对其进行处理，或是中途退出。

# 2. 为什么需要责任链模式？

当处理一个请求、但处理步骤有多种、且依赖于不同的条件时，一种最直观的写法可能是一个巨大的 `if-else` 结构：

```go
func handle(request *Request) {
    if conditionA(request) {
        // 处理 A
    } else if conditionB(request) {
        // 处理 B
    } else if conditionC(request) {
        // 处理 C
    } else {
        // 默认处理
    }
}
```

这种代码的问题显而易见，每当需要增加一种新的处理方式，都必须修改 `handle` 函数，风险很高。

责任链模式就是为了解决这些问题而生的。它将这些处理逻辑拆分成独立的**处理者**对象，并将它们链接成一条**链**。当请求到来时，它会沿着链传递，直到处理流程结束为止。

**核心收益：**
1.  **发送者与接收者解耦**：发送者只需要知道链的第一个处理者，而不需要关心请求最终由谁处理，也不需要知道链的结构。
2.  **灵活性高**：你可以在客户端中随时增加、删除或重新排列链中的处理者，而无需改动现有处理者的代码。
3.  **符合单一职责原则**：每个处理者都只关心自己的处理逻辑，代码更清晰、更易于维护。

每个`Handler`之间的调用关系有点类似于组合模式，但是责任链模式是**链式调用**，而组合模式是**树形结构**。

# 3. 责任链模式的实现 (Go)

首先需要定义`handler`接口，它包含一个方法`Execute`，用于处理请求，以及一个方法`SetNext`，用于设置下一个处理者。

以及具体的处理者实现：

```go
// 处理者接口
type Handler interface {
    Execute(p *client)
    SetNext(next *Handler)
}

// 具体的处理者实现
type Handler1 struct {
    next *Handler
}

// 具体的处理者实现
type Handler2 struct {
    next *Handler
}

func (h *Handler1) Execute(p *client) {
    //...Handle 1处理逻辑，然后调用下一个处理者
    if h.next != nil {
        h.next.Execute(p)
    }
}

func (h *Handler1) SetNext(next *Handler) {
    h.next = next
}

func (h *Handler2) Execute(p *client) {
    //...Handle 2处理逻辑，然后调用下一个处理者
    if h.next != nil {
        h.next.Execute(p)
    }
}

func (h *Handler2) SetNext(next *Handler) {
    h.next = next
}
```

若客户端处理流程为：

```text
client -> Handler1 -> Handler2->over
```

则

```go
h1 := &Handler1{}
h2 := &Handler2{}
h1.SetNext(h2)
c := &client{}
h1.Execute(c)
```

类图:

{% mermaid %}
classDiagram
    class Handler {
        <<interface>>
        +Execute(p *client)
        +SetNext(next *Handler)
    }

    class Handler1 {
        -next: *Handler
        +Execute(p *client)
        +SetNext(next *Handler)
    }

    class Handler2 {
        -next: *Handler
        +Execute(p *client)
        +SetNext(next *Handler)
    }

    class client {
        //...
    }

    Handler <|.. Handler1
    Handler <|.. Handler2

    Handler1 "1" o--> "1" Handler2 : next
    client "1" <.. "1" Handler1 : Execute
{% endmermaid %}


