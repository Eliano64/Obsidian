---
title: prototype
tags:
  - PrototypePattern
  - InterfaceOrientedProgramming
categories:
  - Design Pattern
date: 2025-10-17 00:00:00
katex: true
---

# 1. 什么是原型模式？

一言以蔽之，原型模式（Prototype Pattern）是一种创建型设计模式，它允许客户端直接调用`Clone`方法来创建一个与已有实例完全一致的新的实例，而不需要知道已有实例的构造函数以及构造参数。

# 2. 为什么需要原型模式？

正如上述，原型模式的主要优势在于它允许客户端直接调用`Clone`方法来创建一个与已有实例完全一致的新的实例，而不需要知道已有实例的构造函数以及构造参数。这使得原型模式在需要快速创建多个相同对象的场景下非常有用，例如在游戏开发中创建多个相同类型的敌人实例。

# 3. 原型模式的实现（go）

原型模式提供`prototype`接口，主要需要实现`Clone`方法，该方法返回一个与已有实例完全一致的新的实例。

中间件代码示例：
```go
type prototype interface {
    Clone() prototype
}

type concreteType struct {
    // ...构造参数
}

func (c *concreteType) Clone() prototype {
    return &concreteType{
        // ...构造参数
    }   
}
```

客户端使用：
```go
func main() {
    // 创建一个原型实例
    instance := &concreteType{
        // ...构造参数
    }

    // 使用Clone方法创建一个新的实例
    newInstance := &instance.Clone().(concreteType)
}
```

类图：
{% mermaid %}
classDiagram
class prototype {
    <<interface>>
    +Clone() prototype
}
class concreteType {
    // ...构造参数
}
prototype <|.. concreteType
{% endmermaid %}
