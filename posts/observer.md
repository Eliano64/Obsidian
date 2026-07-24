---
title: observer
tags:
  - ObserverPattern
  - InterfaceOrientedDesign
categories:
  - Design Pattern
date: 2025-11-14 00:00:00
katex: true
---

# 1. 什么是观察者模式？
观察者模式（Observer Pattern）用于在对象状态变化时，通知依赖它的多个观察者，实现一种一对多的订阅关系。被观察者只负责发布事件，不关心有多少观察者、它们如何处理，达到发布方与订阅方的解耦。

# 2. 为什么需要观察者模式？
当一个对象的变化需要联动多个组件时，若直接在对象内部逐个调用这些组件，将产生紧耦合和复杂的依赖。当组件增删或行为变化时，被观察者也不得不调整。通过观察者模式，我们将事件的发布与处理分离：被观察者只负责发出信号，观察者各自处理，从而降低耦合、提高扩展性。

# 3. 观察者模式的实现（go）
```go
type Observer interface {
    Update(event string, data any)
}

type Subject interface {
    Register(o Observer)
    Unregister(o Observer)
    Notify(event string, data any)
}

type Topic struct {
    observers []Observer
}

func (t *Topic) Register(o Observer) {
    t.observers = append(t.observers, o)
}

func (t *Topic) Unregister(o Observer) {
    i := -1
    for idx, ob := range t.observers {
        if ob == o {
            i = idx
            break
        }
    }
    if i >= 0 {
        t.observers = append(t.observers[:i], t.observers[i+1:]...)
    }
}

func (t *Topic) Notify(event string, data any) {
    for _, o := range t.observers {
        o.Update(event, data)
    }
}

type PrintObserver struct {
    name string
}

func (p *PrintObserver) Update(event string, data any) {
    fmt.Printf("[%s] %s: %v\n", p.name, event, data)
}
```

客户端代码如下：

```go
func main() {
    t := &Topic{}
    a := &PrintObserver{name: "A"}
    b := &PrintObserver{name: "B"}

    t.Register(a)
    t.Register(b)

    t.Notify("price_changed", 42)
    t.Unregister(a)
    t.Notify("price_changed", 43)
}
```

输出：

```
[A] price_changed: 42
[B] price_changed: 42
[B] price_changed: 43
```

类图：

{% mermaid %}
classDiagram
class subject {
    <<interface>>
    +Register(o observer)
    +Unregister(o observer)
    +Notify(event,data)
}
class concreteSubject {
    +Register(o observer)
    +Unregister(o observer)
    +Notify(event,data)
}
class observer {
    <<interface>>
    +Update(event,data)
}
class concreteObserver {
    +Update(event,data)
}

subject <|.. concreteSubject
observer <|.. concreteObserver
concreteSubject o--> observer : manages
{% endmermaid %}

