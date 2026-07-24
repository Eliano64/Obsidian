---
title: command
tags:
  - CommandPattern
  - InterfaceOrientedProgramming
categories:
  - Design Pattern
date: 2025-11-08 00:00:00
katex: true
---

# 1. 什么是命令模式

命令模式是一种行为设计模式，它将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化。简单来说，就是将**请求的发送者**和**请求的具体操作**完全解耦。

# 2. 为什么需要命令模式？

命令模式的核心价值在于**解耦**。如果没有命令模式，调用者（比如一个按钮 `Button`）需要直接引用并调用一个具体的操作（比如一个文档 `Document` 的 `save()` 方法）。这会导致 `Button` 和 `Document` 紧密耦合，`Button` 无法被复用到其他操作上。

通过引入命令对象，调用者不再关心命令的接收者是谁，也不关心操作如何执行。它只知道“我需要执行这个命令”。

这样的话，调用者和接收者之间没有任何直接引用，各自可以独立变化。而且添加新的命令非常容易，只需创建新的具体命令类，而无需修改现有代码，符合开放/封闭原则。

# 3. 命令模式的实现 (Go)

中间件：

```go
// Command 接口定义了一个执行操作的方法
type Command interface {
    Execute(receiver *Receiver)
}
// Receiver 是请求的接收者，它知道如何执行与请求相关的操作
type Receiver struct {
    //...
}

func (r *Receiver) Action() {
    // 执行具体的操作
}

// Command1 是一个具体的命令类，它实现了 Command 接口
type Command1 struct {
    //...
}

func (c *Command1) Execute(r *Receiver) {
    r.Action()
}

// Invoker 是请求的发送者，它持有一个 Command 对象  
type Invoker struct {
    command *Command
}

func (i *Invoker) SetCommand(command *Command) {
    i.command = command
}

// ExecuteCommand 是 Invoker 用来间接执行命令的方法
func (i *Invoker) ExecuteCommand(r *Receiver) {
    i.command.Execute(r)
}
```

客户端

```go
receiver := &Receiver{}
command := &Command1{}
invoker := &Invoker{}
invoker.SetCommand(command)
invoker.ExecuteCommand(receiver)
```

类图：
{% mermaid %}
classDiagram
    class Invoker {
        -command: *Command
        +SetCommand(command: *Command)
        +ExecuteCommand(receiver: *Receiver)
    }
    class Command {
        <<interface>>
        +Execute(receiver: *Receiver)
    }
    class Command1 {
        +Execute(receiver: *Receiver)
    }
    class Receiver {
        //...
        +Action()
    }
    Invoker "*" o--> "*" Command: holds
    Command1  ..>  Receiver: executes
    Command1 ..|> Command : implements
{% endmermaid %}