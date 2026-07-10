---
title: singleton
tags:
  - InterfaceOrientedProgramming
  - DesignPattern
  - SingletonPattern
categories:
  - Design Pattern
date: 2025-08-24 00:00:00
katex: true
---

# 1. 什么是单例模式？

单例模式（Singleton Pattern）是一种创建型设计模式，它确保一个类只有一个实例，并提供一个全局访问点来访问该实例。

# 2. 为什么需要单例模式？

在多线程环境下，我们需要多个线程使用同一份资源。

## 2.1 为什么不使用全局变量？

在一些较为简单的环境中，直接使用一些全局变量供所有线程使用是很好的。

但是，如果这个对象是资源密集型的，使用全局变量会导致资源的浪费。而且，简单的全局变量没有提供访问控制（封装），线程可以任意地访问和修改。

而单例模式就可以实现按需创建实例，避免了资源的浪费。而且提供了类封装，确保不会有意外的修改。

# 3. 单例模式的实现（go）

单例模式尤其要注意线程安全。 `sync.Once` 确保了实例只被创建一次，避免了多线程环境下的竞态条件。

以下是一个懒加载的版本。

中间件代码：
```go
var once sync.Once

type singleton struct {
}

var singleInstance *singleton

func GetInstance() *singleton {
    if singleInstance == nil {
        once.Do(
            func() {
                singleInstance = &singleton{}
            })
    } 

    return singleInstance
}
```

客户端代码：
```go
func main() {
    for i := 0; i < 10; i++ {
       go func() {
           instance := GetInstance() // 每个goroutine都获取到相同的实例
       }()
    }
}
```



类图：

{% mermaid %}
classDiagram
    class singleton {
        +GetInstance() singleton
    }
{% endmermaid %}