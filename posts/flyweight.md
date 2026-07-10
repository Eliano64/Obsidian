---
title: flyweight
tags:
  - DesignPattern
  - FlyweightPattern
categories:
  - Design Pattern
date: 2025-10-31 00:00:00
katex: true
---

# 1. 什么是享元模式

享元模式是一种结构型设计模式，它通过共享相同状态的对象来减少内存占用和提高性能。

# 2. 为什么需要享元模式

我们用一个“在地图上种树”的例子来理解。

## 2.1. 问题：内存爆炸

假设要在地图上种 100 万棵树。每棵树都有：
- **坐标**：每棵树都不同。
- **类型信息**：如名称（松树）、颜色（绿色）、纹理图片等。类型是有限的，比如只有几种。

如果为每棵树都创建一个包含所有信息的对象，`Texture`（纹理图片）这份可能很大的数据会被重复存储 100 万次，导致内存爆炸。

```go
// 传统方式：每个对象都包含所有数据
type Tree struct {
	X, Y      int
	Name      string
	Color     string
	Texture   []byte // 假设这份数据很大
}
```

*该如何解决这个问题？*

## 2.2. 方案：享元模式

享元模式的核心是**分离并共享相似对象中的共同部分**，以减少内存占用。

我们将对象的状态分为两种：
- **内部状态 (Intrinsic State)**：可共享、不随环境改变的部分。在例子中是 `Name`, `Color`, `Texture`。
- **外部状态 (Extrinsic State)**：不可共享、随环境改变的部分。在例子中是 `X`, `Y` 坐标。

实现步骤是创建**享元对象**（内部状态），通过**享元工厂**来复用它，并让**上下文对象**（外部状态）持有对享元的引用。

```go
// 1. 享元对象：存储共享的【内部状态】
type treeType struct {
	Name    string
	Color   string
	Texture []byte // 这份数据被共享
}

// 2. 享元工厂：确保相同类型的享元只创建一次
type treeTypeFactory struct {
	types map[string]*treeType
}

func (f *treeTypeFactory) GetTreeType(name, color string, texture []byte) *treeType {
	key := name + "_" + color
	if t, ok := f.types[key]; ok {
		return t // 已存在，直接返回
	}
	// 不存在，创建新的并存起来
	newType := &treeType{name, color, texture}
	f.types[key] = newType
	return newType
}

// 3. 上下文对象：存储【外部状态】和对享元的引用
type Tree struct {
	X, Y int
	Type *treeType // 指向共享的享元对象
}

type Forest struct {
	trees   []*Tree
	factory *treeTypeFactory
}

func (f *Forest) SetFactory() {
	f.factory = &treeTypeFactory{
		types: make(map[string]*treeType),
	}
}

func (f *Forest) PlantTree(x, y int, name, color string, texture []byte) {
	// 从工厂获取共享的 treeType
	treeType := f.factory.GetTreeType(name, color, texture)
	// 创建 Tree，只存储外部状态和引用
	tree := &Tree{X: x, Y: y, Type: treeType}
	f.trees = append(f.trees, tree)
}

func (f *Forest) GetTrees() []*Tree {
	return f.trees
}

```

通过这种方式，`TreeType` 对象（比如“松树”）只会在内存中存在一份，所有松树实例都共享它。内存占用问题迎刃而解。

> - *对于某个类 A 的一些在不同实例中基本相同的属性，我们把它们单独拿出来形成一个类 B，并由一个**缓存池（享元工厂）**来管理。*
> - *新建 A 时，先在缓存池中查找是否已有相同的 B 实例，如果有就复用它，否则创建新的并放入池中。*
> - *最终表现为：多个 A 实例共享同一个 B 对象。*

# 3. 享元模式的实现（go）

如上。

类图：

{% mermaid %}
classDiagram
    class Forest {
        -trees: []*Tree
        -factory: *treeTypeFactory
        +SetFactory()
        +PlantTree(x, y, name, color, texture)
        +GetTrees(): []*Tree
    }
    class treeTypeFactory {
        -types: map[string]*treeType
        +GetTreeType(name, color, texture): *treeType
    }
    class Tree {
        +X: int
        +Y: int
        +Type: *treeType
    }
    class treeType {
        <<Flyweight>>
        +Name: string
        +Color: string
        +Texture: []byte
    }

    Forest "1" o--> "1" treeTypeFactory : uses
    Forest "1" o--> "0..*" Tree : contains
    Tree "1" o-- "1" treeType : references
    treeTypeFactory "1" ..> "0..*" treeType : creates & manages
{% endmermaid %}
