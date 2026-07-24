---
title: memento
tags:
  - MementoPattern
categories:
  - Design Pattern
date: 2025-11-14 00:00:00
katex: true
---

# 1. 什么是备忘录模式？
备忘录模式（Memento Pattern）用于在不破坏封装的前提下，捕获并保存对象的内部状态，以便之后将其恢复。核心思想是：状态快照应对外保持不透明，只有创建它的对象才能读写其内容。

# 2. 为什么需要备忘录模式？
在需要撤销/重做、事务性修改或阶段性试错的场景中，我们希望随时回到某个稳定的历史状态。如果将内部状态直接暴露给外部组件，会破坏封装并引入脆弱耦合。备忘录提供了一种更稳妥的做法：外部只负责保存与传递快照，而不窥视其内容；状态的读写权严格归属于原始对象本身。

# 3. 备忘录模式的实现（go）
下面的实现采用“封装最严格”的变体：看护者只能看到窄接口 `Memento`，无法读取或修改状态；只有发起者可以访问具体备忘录以进行恢复。

```go

type Memento interface{ mementoMarker() }

type memento struct{
    content string
    cursor  int
}

func (m *memento) mementoMarker(){}

type Editor struct{
    content string
    cursor  int
}

func (e *Editor) SetContent(s string){ e.content = s }
func (e *Editor) SetCursor(pos int){ e.cursor = pos }
func (e *Editor) Save() Memento { return &memento{content: e.content, cursor: e.cursor} }
func (e *Editor) Restore(mem Memento){ mm := mem.(*memento); e.content, e.cursor = mm.content, mm.cursor }

type History struct{ stack []Memento }

func (h *History) Push(m Memento){ h.stack = append(h.stack, m) }
func (h *History) Pop() {
    if len(h.stack) == 0 { return }
    i := len(h.stack)-1
    m := h.stack[i]
    h.stack = h.stack[:i]
}
func (h *History) Top() Memento{
    if len(h.stack) == 0 { return nil }
    return h.stack[len(h.stack)-1]
}
```

客户端代码如下：

```go
func main(){
    e := &Editor{}
    h := &History{}

    e.SetContent("a"); e.SetCursor(1); h.Push(e.Save())
    e.SetContent("ab"); e.SetCursor(2); h.Push(e.Save())
    e.SetContent("abc"); e.SetCursor(3)

    h.Pop()
    m := h.Top()
    e.Restore(m)
    fmt.Println(e.content, e.cursor)
}
```

输出：

```
ab 2
```

类图：

{% mermaid %}
classDiagram
class originator {
    +Save() Memento
    +Restore(m Memento)
}
class memento {
    <<interface>>
}
class concreteMemento {
    -state
}
class caretaker {
    +Push(Memento)
    +Pop() Memento
    +Top() Memento
}

originator ..> memento : Save/Restore
memento <|.. concreteMemento
caretaker o-- memento : stores
{% endmermaid %}

