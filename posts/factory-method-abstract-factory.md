---
title: factory method abstract factory
tags:
  - InterfaceOrientedProgramming
  - FactoryMethod
  - AbstractFactory
categories:
  - Design Pattern
date: 2025-10-03 00:00:00
katex: true
---

# 1. 工厂方法

以点菜为例。

```go
type dish interface{ Cook() }

type pizza struct{}
type burger struct{}

func (*pizza) Cook() {  }
func (*burger) Cook() {  }
```

若不用工厂方法，则客户端直连产品，新增产品要改分支，不符合开闭原则。

```go
func orderDish(typeName string) dish {
    switch typeName {
    case "pizza":
        d := &pizza{}
        d.Cook() // "cook pizza"
        return d
    case "burger":
        d := &burger{}
        d.Cook() // "cook burger"
        return d
    default:
        return nil
    }
}
```
而使用工厂方法后，客户端只依赖抽象工厂，新增产品时只需要新增具体工厂即可，符合开闭原则。

即，**把“怎么做菜”的细节从用户（软件开发程序员）手里拿走，交给各自的工厂（中间件）。**

```go
type Factory interface {
    Create() dish
}

type PizzaFactory struct{}
func (PizzaFactory) Create() dish { return &pizza{} }

type BurgerFactory struct{}
func (BurgerFactory) Create() dish { return &burger{} }

func order(factory Factory) dish {
    d := factory.Create()
    d.Cook() 
    return d
}
```

这样的话，新增产品时只需要新增产品 + 工厂即可，无需修改用户程序。符合开闭原则。

```go
type sushi struct{}
func (s *sushi) Cook() {  }

type SushiFactory struct{}
func (SushiFactory) Create() dish { return &sushi{} }

```

客户端只依赖抽象工厂，只需调用工厂方法即可，不用修改。

类图：

{% mermaid %}
classDiagram
  class Dish {
    +Cook() 
  }
  class Pizza {
    +Cook() 
  }
  class Burger {
    +Cook() 
  }
  Dish <|.. Pizza
  Dish <|.. Burger

  class Factory {
    +Create() Dish
  }
  class PizzaFactory
  class BurgerFactory
  Factory <|.. PizzaFactory
  Factory <|.. BurgerFactory

  Factory ..> Dish : Create()
{% endmermaid %}

# 2. 抽象工厂

场景：套餐由两个相关组件组成（菜 Dish + 饮料 Drink），同一风格的组件应该一起创建并保持搭配一致（意式：Pizza+Coffee，美式：Burger+Cola）。

```go
// 产品族接口
type dish interface{ Cook() }
type drink interface{ Serve() }

// 具体产品
type pizza struct{}
func (*pizza) Cook() {  }

type burger struct{}
func (*burger) Cook() {  }

type coffee struct{}
func (coffee) Serve() {  }

type cola struct{}
func (cola) Serve() {  }    
```

抽象工厂提供“同时创建多个相关产品”的统一接口，客户端只选择工厂（风格），不再传字符串。

中间件：工厂方法的“实现”，负责创建具体产品。

```go
// 抽象工厂：一次创建同一风格的多个相关产品
type kitchenFactory interface {
    CreateDish() dish
    CreateDrink() drink
}

// 具体工厂：意式（Pizza + Coffee）
type italianFactory struct{}
func (italianFactory) CreateDish() dish   { return &pizza{} }
func (italianFactory) CreateDrink() drink { return &coffee{} }

// 具体工厂：美式（Burger + Cola）
type americanFactory struct{}
func (americanFactory) CreateDish() dish   { return &burger{} }
func (americanFactory) CreateDrink() drink { return &cola{} }
```

用户只需要依赖抽象工厂，无需知道具体产品。


```go
func orderSet(f kitchenFactory) (dish, drink) {
    d := f.CreateDish()
    d.Cook()
    r := f.CreateDrink()
    r.Serve()
    return d, r
}
```

类图：

{% mermaid %}
classDiagram
class dish {
+Cook()
}
class drink {
+Serve()
}

class pizza {
+Cook()
}
class burger {
+Cook()
}
class coffee {
+Serve()
}
class cola {
+Serve()
}

dish <|.. pizza
dish <|.. burger
drink <|.. coffee
drink <|.. cola

class kitchenFactory {
+CreateDish() dish
+CreateDrink() drink
}
class italianFactory
class americanFactory

kitchenFactory <|.. italianFactory
kitchenFactory <|.. americanFactory

italianFactory ..> dish : CreateDish()
italianFactory ..> drink : CreateDrink()
americanFactory ..> dish : CreateDish()
americanFactory ..> drink : CreateDrink()
{% endmermaid %}



- 客户端不写 switch、不传 "pizza/burger" 字符串；只选风格工厂（Italian/American）。
- 搭配关系由工厂保证，防止“风格混搭”造成业务不一致。
- 扩展新风格时，仅新增“具体工厂 + 其产品”，保持开闭原则。