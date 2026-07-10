---
title: decorator
tags:
  - DesignPattern
  - DecoratorPattern
  - InterfaceOrientedDesign
categories:
  - Design Pattern
date: 2025-10-25 00:00:00
katex: true
---

# 1. 什么是装饰器模式

装饰器模式（Decorator Pattern）是一种结构型设计模式，它允许你在不改变原有对象结构的情况下，动态地给对象添加新的功能。

# 2. 为什么需要装饰器模式

想象一下去咖啡店点单的场景：
- 一杯基础款咖啡（比如美式）。
- 各种调料（比如牛奶、糖、巧克力酱等）。

你可以选择只喝纯的美式，也可以要求加奶，再加糖。

如果我们用继承来实现“咖啡+调料”的功能，会发生什么？

我们可能需要创建`CoffeeWithMilk` `CoffeeWithSugar` `CoffeeWithMilkAndSugar`等大量的子类。

如果再增加一种调料（比如奶油），类的数量就会爆炸式增长。

同时，如果想给一个已经创建好的`Coffee`实例动态添加牛奶，使用继承是做不到的，因为继承是静态的。

但装饰器模式通过组合解决了这些问题：

1. 遵循开闭原则：你可以随时创建新的调料（装饰器），而无需修改任何现有的咖啡（组件）代码。系统对扩展开放，对修改关闭。
2. 灵活性和动态性：可以在运行时根据需要，任意地、动态地为对象添加一层或多层功能，组合顺序和数量都非常灵活。
3. 避免子类爆炸：无论有多少种调料，我们只需要为每种调料创建一个装饰器类，而不是为每种组合都创建一个子类。

# 3. 装饰器模式的实现（go）

```go
type Coffee interface {
	GetCoffee() string
}

type Latte struct {
}

func (l *Latte) GetCoffee() string {
	return "Latte"
}

type CoffeeDecorator struct {
    CoffeeOrdered *Coffee
    AddCondiment string
}

func (d *CoffeeDecorator) GetCoffee() string {
	return d.CoffeeOrdered.GetCoffee() + " with " + d.AddCondiment
}
```

客户端
```go
func main() {
	latte := &Latte{}
	latteWithMilk := &CoffeeDecorator{CoffeeOrdered: latte, AddCondiment: "Milk"}
	latteWithMilkAndSugar := &CoffeeDecorator{CoffeeOrdered: latteWithMilk, AddCondiment: "Sugar"}
	fmt.Println(latteWithMilkAndSugar.GetCoffee())
}
```
类图

{% mermaid %}
classDiagram
    class Coffee {
        <<interface>>
        +GetCoffee() string
    }
    class Latte {
        +GetCoffee() string
    }
    class CoffeeDecorator {
        +GetCoffee() string
        +AddCondiment string
        +CoffeeOrdered *Coffee
    }
    Coffee <|.. Latte
    CoffeeDecorator "1" *-- "1" Coffee
{% endmermaid %}