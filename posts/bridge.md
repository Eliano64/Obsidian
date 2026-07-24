---
title: bridge
tags:
  - BridgePattern
  - InterfaceOrientedProgramming
categories:
  - Design Pattern
date: 2025-10-18 00:00:00
katex: true
---

# 1. 什么是桥接模式？

桥接模式（Bridge Pattern）是一种结构型设计模式，它的核心思想是**将抽象部分与它的实现部分分离，使它们都可以独立地变化**。

想象一下电视机和遥控器的关系：
* **抽象层**：遥控器。它定义了高层操作，如 `开机()`、`关机()`、`换台()`。不同的遥控器（如 `普通遥控器`、`智能遥控器`）可以有不同的交互方式，但它们都服务于控制电视这个目的。
* **实现层**：电视机。它负责执行具体的操作，如 `物理开机()`、`切换视频源()`。不同的电视品牌（如 `索尼电视`、`三星电视`）有各自的实现细节。

桥接模式就像是在遥控器中内置一个对电视机实例的引用。遥控器的按钮按下后，会调用它所引用的那台电视机的具体方法。通过这种方式，你可以独立地更换遥控器（从普通遥控器升级到智能遥控器），而无需关心电视机的品牌；同样，你也可以独立地更换电视机（从索尼换成三星），而遥控器无需任何改变。它们通过一个“桥梁”连接，各自演化。

# 2. 为什么需要桥接模式？

当一个类或系统存在**多个独立变化的维度**时，桥接模式就显得尤为重要。如果不使用桥接模式，我们可能会倾向于使用继承来解决问题，但这会导致类的数量爆炸式增长。

设想一个场景：我们要绘制不同颜色（如红色、蓝色）的形状（如圆形、方形）。

*   **维度一**：形状（Circle, Square）
*   **维度二**：颜色（Red, Blue）

如果使用继承，我们可能需要创建 `RedCircle`, `BlueCircle`, `RedSquare`, `BlueSquare` 这样 `2 * 2 = 4` 个类。如果再增加一种形状（三角形）和一种颜色（绿色），类的数量就会变成 `3 * 3 = 9` 个。这种“笛卡尔积”式的类爆炸是难以维护的。

桥接模式通过将“形状”和“颜色”两个维度分离来解决这个问题：
*   **抽象部分**：`Shape` (形状)，它包含一个对 `Color` (颜色) 的引用。
*   **实现部分**：`Color` (颜色)，它是一个接口，有 `Red`、`Blue` 等具体实现。

这样，类的数量就从 `M * N` 变成了 `M + N`。当需要增加一种新形状或新颜色时，只需增加一个新类，而不会影响到其他部分，完美遵循了开闭原则。

# 3. 桥接模式的实现（Go）

下面我们用 Go 来实现上述的“形状与颜色”的例子。

中间件：

```go
// 1. 实现部分 (Implementor)
// Color 是实现部分的接口
type Color interface {
	ApplyColor() string
}

// 2. 具体实现 (Concrete Implementors)
// RedColor 是一个具体的颜色实现
type RedColor struct{}

func (r *RedColor) ApplyColor() string {
	return "Red"
}

// BlueColor 是另一个具体的颜色实现
type BlueColor struct{}

func (b *BlueColor) ApplyColor() string {
	return "Blue"
}

// 3. 抽象部分 (Abstraction)
// Shape 是抽象部分的接口，它包含一个对实现部分(Color)的引用
type Shape interface {
	Draw()
}

// 4. 扩展抽象 (Refined Abstraction)
// Circle 是一个具体的形状，它持有一个 Color 对象
type Circle struct {
	color *Color
}

func (c *Circle) Draw() {
	fmt.Printf("Drawing a Circle in %s color.\n", c.color.ApplyColor())
}

// Square 是另一个具体的形状
type Square struct {
	color *Color
}

func (s *Square) Draw() {
	fmt.Printf("Drawing a Square in %s color.\n", s.color.ApplyColor())
}
```

客户端代码:
```go
func main() {
	// 创建具体的实现对象
	red := &RedColor{}
	blue := &BlueColor{}

	// 创建具体的抽象对象，并将实现对象“桥接”进去
	redCircle := &Circle{color: red}
	blueSquare := &Square{color: blue}

	// 调用方法
	redCircle.Draw()   // 输出: Drawing a Circle in Red color.
	blueSquare.Draw()  // 输出: Drawing a Square in Blue color.
}
```

类图：

{% mermaid %}
classDiagram
    class Shape {
        <<interface>>
        +Draw()
    }
    class Circle {
        -color: Color
        +Draw()
    }
    class Square {
        -color: Color
        +Draw() 
    }
    class Color {
        <<interface>>
        +ApplyColor() string
    }
    class RedColor {
        +ApplyColor() string
    }
    class BlueColor {
        +ApplyColor() string        
    }
    Circle --|> Shape
    Square --|> Shape
    Circle --> Color
    Square --> Color
    Circle "1" *-- "1" Color
    Square "1" *-- "1" Color
    RedColor ..|> Color
    BlueColor ..|> Color
{% endmermaid %}

在这个例子中，`Shape` 和 `Color` 可以独立地进行扩展。如果想增加一个 `Green` 颜色，只需实现 `Color` 接口即可；如果想增加一个 `Triangle` 形状，只需创建一个新的 `Triangle` 结构体并实现 `Draw` 方法，而无需改动任何现有代码。



