---
title: builder
tags:
  - BuilderPattern
  - InterfaceOrientedProgramming
categories:
  - Design Pattern
date: 2025-10-12 00:00:00
katex: true
---

# 1. 什么是生成器模式？

生成器模式（Builder Pattern）是一种创建型设计模式，它将一个复杂对象的构建分成多个步骤，每个步骤可以被重写，因此使得同样的构建过程可以生成不同类型和形式的对象。

# 2. 为什么需要生成器模式？

在一些情况下，我们需要根据不同的需求来创建不同类型的对象。如果使用简单的构造函数，会导致构造函数参数列表过长，或者参数顺序错误的问题。而生成器模式可以将对象的创建过程分解成多个步骤，每个步骤可以根据需求进行定制，从而避免了构造函数参数列表过长的问题。

# 3. 生成器模式的实现（go）

生成器需要两个接口：
- `Builder`接口：是构建对象的接口，定义了构建对象需要哪些方法。
- `Director`接口：定义了使用`Builder`接口来构建对象的过程。

中间件代码示例：
```go
type builder interface {
    SetPartA()
    SetPartB()
    SetPartC() 
    GetProduct() *product
}

type concreteBuilderA struct {}

func (concreteBuilderA) SetPartA() {
    // 实现设置PartA的逻辑
}

func (concreteBuilderA) SetPartB() {
    // 实现设置PartB的逻辑
}

func (concreteBuilderA) SetPartC() {
    // 实现设置PartC的逻辑
}

func (concreteBuilderA) GetProduct() *product {
    // 实现返回产品的逻辑
    return &product{}
}

type concreteBuilderB struct {}
// 实现Builder接口的方法,与concreteBuilderA相同，下略

type product interface {
    //...
}
// 产品A和产品B都是继承自`product`的定义，下略

type productA struct{
    //...
}

type productB struct{
    //...
}

type director struct {
    builder* builder
}

func (d *director) SetBuilder(b builder) {
    d.builder = b
}

func (d *director) Construct() *product {
    d.builder.SetPartA()
    d.builder.SetPartB()
    d.builder.SetPartC()
    return d.builder.GetProduct()
}
```

这样，调用代码如下：
```go
director := &director{}
builderA := &concreteBuilderA{}
director.SetBuilder(builderA)
productA := director.Construct()

builderB := &concreteBuilderB{}
director.SetBuilder(builderB)
productB := director.Construct()
```

类图：

{% mermaid %}
classDiagram
class builder {
    <<interface>>
    +SetPartA()
    +SetPartB()
    +SetPartC() 
    +GetProduct() *product
}
class concreteBuilderA {
    +SetPartA()
    +SetPartB()
    +SetPartC() 
    +GetProduct() *productA
}
class concreteBuilderB {
    +SetPartA()
    +SetPartB()
    +SetPartC() 
    +GetProduct() *productB
}
class director {
    <<interface>>       
    +SetBuilder(b builder)
    +Construct() *product
}

class product {
    <<interface>> 
    //...
}

class productA {
    //...
}
class productB {
    //...
}
builder <|.. concreteBuilderA
builder <|.. concreteBuilderB
product <|.. productA
product <|.. productB
director ..> builder : SetBuilder()
director ..> product : Construct()
director ..> product : Construct()
{% endmermaid %}

比起不停地修改、维护构造函数以及调用，使用生成器模式只需定义好对应的`Builder`接口，统一调用`Director`接口的`Construct`方法即可。
