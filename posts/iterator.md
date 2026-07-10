---
title: iterator
tags:
  - DesignPattern
  - IteratorPattern
  - InterfaceOrientedProgramming
categories:
  - Design Pattern
date: 2025-11-08 00:00:00
katex: true
---

# 1. 什么是迭代器模式

迭代器模式（Iterator Pattern）是一种行为设计模式，它提供一种方法来顺序访问一个聚合对象（Aggregate Object）中的各个元素，而又无需暴露该对象的内部表示。

无论操作的是一个数组、一个链表、一棵树还是一个哈希表，都可以用同样的客户端方式去遍历它（比如一个 `for-each` 循环），而不关心底层的具体的遍历方式是dfs还是bf或者其他。

它将遍历的逻辑从聚合对象中分离出来，放入一个单独的**迭代器**对象中。这样，容器只负责存储数据，而迭代器负责遍历数据。

# 2. 为什么需要迭代器模式？

如果没有迭代器模式，客户端代码需要知道聚合对象的内部结构才能进行遍历。例如，遍历数组需要用索引，遍历链表需要用 `next` 指针。这会导致客户端与数据结构紧密耦合。

* 客户端无需关心容器的内部实现（是 `slice`、`map` 还是 `linked list`），使得更换或修改容器实现变得容易，不影响客户端代码。
* 为不同的数据结构提供了统一的遍历方式，简化了客户端代码。
* 可以为同一个聚合对象提供多种不同的迭代器（如正向遍历、反向遍历）。
* 符合单一职责原则：聚合对象专注于数据存储，而迭代器专注于遍历，职责清晰。

Go 语言的 `for...range` 循环就是迭代器模式的一个典型应用，它为数组、切片、字符串、map 和 channel 提供了统一的遍历语法。

# 3. 迭代器模式的实现 (Go)

> 对一个二叉树实现迭代器模式，要求迭代器能够按**中序遍历**二叉树。

```go
// Iterator 是泛型迭代器接口
type Iterator[E any] interface {
	IsEnd() bool
	GetCur() E
}

// Collection 是泛型聚合接口
type Collection[E any] interface {
	CreateIterator() Iterator[E]
}
// TreeNode
type TreeNode[T any] struct {
	Val   T
	Left  *TreeNode[T]
	Right *TreeNode[T]
}

// BinaryTree 是我们的具体聚合类
type BinaryTree[T any] struct {
	Root *TreeNode[T]
}
```

[InOrder](https://eliano64.github.io/2025/08/24/250824-cogitation-%E4%BA%8C%E5%8F%89%E6%A0%91%E9%81%8D%E5%8E%86%E7%9A%84%E8%BF%AD%E4%BB%A3%E6%B3%95/#2.%20%E4%B8%AD%E5%BA%8F%E9%81%8D%E5%8E%86)

```go
// InOrderIterator 是具体迭代器类
type InOrderIterator[T any] struct {
	stack []*TreeNode[T]
}

// CreateIterator 是 BinaryTree 实现 Collection 接口的方法
func (bt *BinaryTree[T]) CreateIterator() Iterator[*TreeNode[T]] {
	it := &InOrderIterator[T]{stack: make([]*TreeNode[T], 0)}
	// 初始化时，将根节点及其所有左子节点压入栈
	it.pushLeft(bt.Root)
	return it
}

// pushLeft 辅助函数，将一个节点及其所有左子节点压入栈
func (it *InOrderIterator[T]) pushLeft(node *TreeNode[T]) {
	for node != nil {
		it.stack = append(it.stack, node)
		node = node.Left
	}
}

func (it *InOrderIterator[T]) IsEnd() bool {
	return len(it.stack) <= 0
}

// GetCur 返回当前迭代器指向的节点，且将迭代器指向下一个节点
func (it *InOrderIterator[T]) GetCur() *TreeNode[T] {
    if(it.IsEnd()){
        return nil
    }
	// 从栈顶弹出一个节点
	n := len(it.stack) - 1
	node := it.stack[n]
	it.stack = it.stack[:n]

	// 如果该节点有右子树，则将右子节点及其所有左子节点压栈
	if node.Right != nil {
		it.pushLeft(node.Right)
	}
	return node
}
```

```go
func main() {
	// 构造一棵二叉树
	//      4
	//     / \
	//    2   5
	//   / \
	//  1   3
	root := &TreeNode[int]{
		Val: 4,
		Left: &TreeNode[int]{
			Val: 2,
			Left:  &TreeNode[int]{Val: 1},
			Right: &TreeNode[int]{Val: 3},
		},
		Right: &TreeNode[int]{Val: 5},
	}

	// 泛型客户端通过接口使用
	c := &BinaryTree[int]{Root: root}
	it := c.CreateIterator() // Iterator[*TreeNode[int]]

	for !it.IsEnd() {
		fmt.Printf("%d ", it.GetCur().Val)
	}
	// 输出: 1 2 3 4 5
}
```

类图：

{% mermaid %}
classDiagram
    class Iterator {
        <<interface>>
        +IsEnd() bool
        +GetCur() E
    }
    class Collection {
        <<interface>>
        +CreateIterator() Iterator[E]
    }
    class TreeNode {
        +Val T
        +Left *TreeNode[T]
        +Right *TreeNode[T]
    }
    class BinaryTree {
        +Root *TreeNode[T]  
    }
    class InOrderIterator {
        -stack []*TreeNode[T]
        -pushLeft(node *TreeNode[T])
        +IsEnd() bool
        +GetCur() *TreeNode[T]+
    }
    Iterator <|.. InOrderIterator
    Collection <|.. BinaryTree
    TreeNode "1" <--* "*" BinaryTree : node
    Collection ..> Iterator : Creates
{% endmermaid %}