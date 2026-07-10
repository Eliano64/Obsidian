---
title: adapter
tags:
  - DesignPattern
  - AdapterPattern
  - InterfaceOrientedProgramming
categories:
  - Design Pattern
date: 2025-10-18 00:00:00
katex: true
---

# 1. 什么是适配器模式？

适配器模式（Adapter Pattern）是一种结构型设计模式，它允许将一个类的接口转换为客户端期望的另一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的类可以协同工作。

就像 “USB-C 转 3.5mm 音频” 转换器。很多新款手机（被适配者 Adaptee）取消了传统的 3.5mm 耳机孔，只提供 USB-C 接口。如果你想使用旧的 3.5mm 插头耳机（客户端 Client），就会遇到接口不兼容的问题。

此时，这个转换器就充当了适配器（Adapter）的角色。它将手机的 USB-C 接口转换为耳机（客户端）所期望的 3.5mm 音频插孔。通过这个小小的转换器，耳机无需任何改动，就能在新手机上继续使用，适配器完美地弥合了两者之间的接口差异。

# 2. 为什么需要适配器模式？

在软件开发中，我们经常会遇到以下几种情况，这时适配器模式就显得尤为重要：

1.  **复用现有代码**：当一个已存在的类功能很强大，能满足你的大部分需求，但它的接口与你当前系统要求的接口不一致时。重写这个类会耗费大量时间且容易引入错误，而使用适配器则可以在不改变原有代码的基础上，将其“包装”成你需要的样子。

2.  **集成第三方库或遗留系统**：第三方库或老旧系统的接口通常无法修改。如果它们的接口与你的系统不兼容，适配器可以作为中间层，将这些外部接口转换为系统内部统一的、标准的接口，从而实现无缝集成。

3.  **解耦客户端与具体实现**：客户端代码只依赖于一个标准的目标接口（Target），而不需要关心背后真正干活的类（Adaptee）是谁，它的接口长什么样。这样一来，当底层的具体实现需要更换时，只需要更换或修改对应的适配器，而客户端代码完全不受影响，增强了系统的灵活性和可维护性。

总而言之，适配器模式遵循了“开闭原则”（对扩展开放，对修改关闭），它允许我们在不修改现有代码的情况下，通过增加新的适配器类来扩展系统的功能。

# 3. 适配器模式的实现（go）

假设我们有一个音乐播放器 `MusicPlayer`，它依赖一个 `Player` 接口来播放音乐。

```go
// Player是我们期望的目标接口 (Target)
type Player interface {
    PlayMusic()
}
```

现在，我们有一个现成的、无法修改的第三方音频库 `AwesomePlayer`，但它的播放方法是 `PlaySound`，与我们的 `Player` 接口不兼容。

```go
// AwesomePlayer 是一个已存在的、接口不兼容的类 (Adaptee)
type AwesomePlayer struct{}

func (p *AwesomePlayer) PlaySound() {
    fmt.Println("Playing sound with AwesomePlayer")
}
```

为了让 `MusicPlayer` 能使用 `AwesomePlayer`，我们创建一个适配器 `AwesomePlayerAdapter`。

```go
// AwesomePlayerAdapter 是适配器 (Adapter)
type AwesomePlayerAdapter struct {
    adaptee *AwesomePlayer // 持有被适配的对象
}

// NewAwesomePlayerAdapter 创建一个新的适配器实例
func NewAwesomePlayerAdapter(adaptee *AwesomePlayer) *AwesomePlayerAdapter {
    return &AwesomePlayerAdapter{adaptee: adaptee}
}

// PlayMusic 实现了 Player 接口，内部调用被适配者的 PlaySound 方法
func (a *AwesomePlayerAdapter) PlayMusic() {
    a.adaptee.PlaySound()
}
```

现在，客户端代码可以像这样使用：

```go
func main() {
    // 创建被适配的对象
    awesomePlayer := &AwesomePlayer{}
    
    // 创建适配器，并将其传入
    var player Player = NewAwesomePlayerAdapter(awesomePlayer)
    
    // 客户端通过统一的 PlayMusic 接口进行播放，无需关心底层实现
    player.PlayMusic() // 输出: Playing sound with AwesomePlayer
}
```

在这个例子中，`AwesomePlayerAdapter` 成功地将 `AwesomePlayer` 的 `PlaySound` 接口转换为了 `Player` 接口所期望的 `PlayMusic` 接口，使得客户端可以无缝地使用这个第三方库。

类图：

{% mermaid %}
classDiagram
    class Player {
        <<interface>>
        +PlayMusic()
    }
    class AwesomePlayer {
        +PlaySound()
    }
    class AwesomePlayerAdapter {
        -adaptee: AwesomePlayer
        +PlayMusic()
    }
    Player <|.. AwesomePlayerAdapter
    AwesomePlayerAdapter "1" *-- "1" AwesomePlayer
    AwesomePlayerAdapter ..> AwesomePlayer : PlayMusic() call PlaySound()
{% endmermaid %}