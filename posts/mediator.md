---
title: mediator
tags:
  - DesignPattern
  - MediatorPattern
categories:
  - Design Pattern
date: 2025-11-14 00:00:00
katex: true
---

# 1. 什么是中介者模式？
中介者模式（Mediator Pattern）通过引入一个“中介者”对象来封装对象之间的交互，使多个对象不需要显式引用彼此，改为只与中介者通信。它将协作编排从同事对象中抽离出来，降低耦合并集中管理交互规则。

# 2. 为什么需要中介者模式？

一个复杂的软件系统，若每个对象都与其他许多对象直接关联，则它们之间产生了紧密的耦合。这种设计的后果是，对任何一个对象的修改都可能引发连锁反应，迫使其他相关对象也得跟着改动，这便是“牵一发而动全身”的困境。系统的维护成本因此急剧攀升。

更糟糕的是，完整的业务流程被切割得支离破碎，散落在各个对象的实现中。想要理解一个完整的协作过程，开发者不得不深入多个类，追踪复杂的调用链，这使得系统的整体行为变得难以把握，也让新功能的添加或问题排查变得困难。同时，由于每个对象都和它的“同事们”绑得太死，想把它单独拿出来在别处复用也几乎不可能。

中介者模式正是为了解开这个“死结”而提出的。它引入了一个中心协调者——“中介者”，所有对象不再互相直接通信，而是统一与中介者对话。中介者负责接收消息，并根据既定规则将其分发给正确的对象。这样，原本混乱的网状结构被重构为清晰的“星型结构”，所有复杂的交互逻辑都集中到了中介者内部。通过这种方式，对象之间得以解耦，系统也因此变得更加清晰、易于维护和扩展。

# 3. 中介者模式的实现（go）
下面用一个简化的聊天室演示中介者如何协调用户之间的消息传递。

```go
package main

import (
    "fmt"
)

type Mediator interface {
    Register(c Colleague)
    Send(from string, to string, msg string)
    Broadcast(from string, msg string)
}

type Colleague interface {
    Name() string
    Receive(from string, msg string)
    Send(to string, msg string)
    Broadcast(msg string)
}

type ChatRoom struct {
    members map[string]Colleague
}

func NewChatRoom() *ChatRoom {
    return &ChatRoom{members: make(map[string]Colleague)}
}

func (r *ChatRoom) Register(c Colleague) {
    r.members[c.Name()] = c
}

func (r *ChatRoom) Send(from string, to string, msg string) {
    if m, ok := r.members[to]; ok {
        m.Receive(from, msg)
    }
}

func (r *ChatRoom) Broadcast(from string, msg string) {
    for name, m := range r.members {
        if name == from {
            continue
        }
        m.Receive(from, msg)
    }
}

type User struct {
    name string
    room *ChatRoom
}

func NewUser(name string, room *ChatRoom) *User {
    return &User{name: name, room: room}
}

func (u *User) Name() string { return u.name }

func (u *User) Receive(from string, msg string) {
    fmt.Printf("[%s] <- %s: %s\n", u.name, from, msg)
}

func (u *User) Send(to string, msg string) {
    u.room.Send(u.name, to, msg)
}

func (u *User) Broadcast(msg string) {
    u.room.Broadcast(u.name, msg)
}

func main() {
    room := NewChatRoom()

    alice := NewUser("alice", room)
    bob := NewUser("bob", room)
    eve := NewUser("eve", room)

    room.Register(alice)
    room.Register(bob)
    room.Register(eve)

    alice.Send("bob", "Hi Bob")
    bob.Send("alice", "Hi Alice")
    eve.Broadcast("Hello everyone")
}
```

要点：用户之间不直接互相引用，交互规则集中在 `ChatRoom`。当需要增加记录审计、敏感词过滤、分组路由时，仅需扩展中介者。

类图：

{% mermaid %}
classDiagram
class mediator {
    <<interface>>
    +Register(c colleague)
    +Send(from,to,msg)
    +Broadcast(from,msg)
}
class concreteMediator {
    +Register(c colleague)
    +Send(from,to,msg)
    +Broadcast(from,msg)
}
class colleague {
    <<interface>>
    +Name() string
    +Receive(from,msg)
    +Send(to,msg)
    +Broadcast(msg)
}
class concreteColleague {
    +Name() string
    +Receive(from,msg)
    +Send(to,msg)
    +Broadcast(msg)
}

mediator <|.. concreteMediator
colleague <|.. concreteColleague
concreteMediator --> colleague : Register()
concreteColleague --> mediator : Send()/Broadcast()
concreteMediator --> concreteColleague : Send()/Broadcast()
{% endmermaid %}
