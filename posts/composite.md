---
title: composite
tags:
  - CompositePattern
  - InterfaceOrientedDesign
categories:
  - Design Pattern
date: 2025-10-25 00:00:00
katex: true
---

# 1. 什么是组合模式

组合模式（Composite Pattern）是一种结构型设计模式，它允许你将对象组合成树形结构以表示“部分-整体”的层次关系。组合模式使得客户端可以统一地处理个别对象和对象组合。

# 2. 为什么需要组合模式

组合最主要的功能是在整个树状结构上递归调用方法并对结果进行汇总。

# 3. 组合模式的实现（go）

假定一个文件系统，其中包含文件和目录。目录可以包含其他文件和目录。

现在我们需要搜索文件系统中是否包含某个文件。

```go
type Component interface {
	Search(keyword string)
}

type Directory struct {
	Name     string
	children []Component
}

func (d *Directory) Search(keyword string) {
	if d.Name == keyword {
		fmt.Println("find directory:", d.Name)
		return
	}
	for _, child := range d.children {
		child.Search(keyword)
	}
}

// add 方法用于添加子组件
func (d *Directory) Add(child Component) {
	d.children = append(d.children, child)
}

type File struct {
	Name string
}

func (f *File) Search(keyword string) {
	if f.Name == keyword {
		fmt.Println("find file:", f.Name)
		return
	}
}
```

客户端代码

```go
func main() {
    root := &Directory{Name: "root"}
    dir1 := &Directory{Name: "dir1"}
    dir2 := &Directory{Name: "dir2"}
    file1 := &File{Name: "file1"}
    file2 := &File{Name: "file2"}
    root.Add(dir1)
    root.Add(dir2)
    dir1.Add(file1)
    dir2.Add(file2)

    root.Search("file1")
    root.Search("file2")
    root.Search("dir1")
    root.Search("dir2")
}   
```

类图

{% mermaid %}
classDiagram
    class Component {
        <<interface>>
        +Search(keyword string)
    }
    class Directory {
        +Name string
        -children []Component
        +Add(child Component)
        +Search(keyword string)
    }
    class File {
        +Name string
        +Search(keyword string)
    }
    Component <|.. Directory
    Component <|.. File
    Directory "1" o-- "n" Component
{% endmermaid %}
