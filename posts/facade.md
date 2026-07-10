---
title: facade
tags:
  - FacadePattern
  - DesignPattern
  - InterfaceOrientedDesign
categories:
  - Design Pattern
date: 2025-10-25 00:00:00
katex: true
---

# 1. 什么是外观模式

外观模式是一种结构型设计模式， 能为程序库、 框架或其他复杂类提供一个简单的接口。

# 2. 为什么需要外观模式

外观模式的主要目的是简化复杂系统的使用。它通过提供一个简单的接口，隐藏系统的复杂性，使客户端代码更易于理解和使用。

# 3. 外观模式的实现（go）

以计算机开机为例，展示外观模式的实现。

```go
// 子系统：CPU
type CPU struct{}

func (c *CPU) Start()    { fmt.Println("CPU: start") }
func (c *CPU) Shutdown() { fmt.Println("CPU: shutdown") }

// 子系统：内存
type Memory struct{}

func (m *Memory) LoadOS() { fmt.Println("Memory: load OS into memory") }
func (m *Memory) Clear()  { fmt.Println("Memory: clear") }

// 子系统：磁盘
type Disk struct{}

func (d *Disk) ReadBootSector() { fmt.Println("Disk: read boot sector") }

// 外观：计算机
type ComputerFacade struct {
	cpu  *CPU
	mem  *Memory
	disk *Disk
}

func NewComputerFacade() *ComputerFacade {
	return &ComputerFacade{
		cpu:  &CPU{},
		mem:  &Memory{},
		disk: &Disk{},
	}
}

// 对外暴露的简化接口：开机流程
func (c *ComputerFacade) PowerOn() {
	fmt.Println("== PowerOn ==")
	c.disk.ReadBootSector()
	c.cpu.Start()
	c.mem.LoadOS()
	fmt.Println("System is up.")
}

// 对外暴露的简化接口：关机流程
func (c *ComputerFacade) PowerOff() {
	fmt.Println("== PowerOff ==")
	c.cpu.Shutdown()
	c.mem.Clear()
	fmt.Println("System is down.")
}
```

客户端

```go
func main() {
	pc := NewComputerFacade()
	pc.PowerOn()
	pc.PowerOff()
}
```

类图

{% mermaid %}
classDiagram
    class CPU {
        +Start()
        +Shutdown()
    }
    class Memory {
        +LoadOS()
        +Clear()
    }
    class Disk {
        +ReadBootSector()
    }
    class ComputerFacade {
        -cpu: CPU
        -mem: Memory
        -disk: Disk
        +NewComputerFacade()
        +PowerOn()
        +PowerOff()
    }
    ComputerFacade *-- CPU : uses
    ComputerFacade *-- Memory : uses
    ComputerFacade *-- Disk : uses
    ComputerFacade ..> CPU : Start()
    ComputerFacade ..> Memory : LoadOS()
    ComputerFacade ..> Disk : ReadBootSector()
    ComputerFacade ..> CPU : Shutdown()
    ComputerFacade ..> Memory : Clear()
{% endmermaid %}


