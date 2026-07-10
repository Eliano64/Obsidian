---
title: Binary Tree Storage 
tags:
  - DataStructureAndAlgorithm
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:39:00 
katex: true
---

# 二叉树的存储结构

## 顺序存储

二叉树顺序存储时，必须把二叉树结点编号与完全二叉树的层序编号对应起来。

若编号从 1 开始：

- 结点 `i` 的左孩子编号为 `2i`。
- 结点 `i` 的右孩子编号为 `2i+1`。
- 结点 `i` 的双亲编号为 `⌊i/2⌋`。

顺序存储适合[[complete-binary-tree|完全二叉树]]。若普通二叉树也强行按完全二叉树位置存储，需要为缺失结点保留空位，可能造成严重浪费。

若编号从 0 开始：

- 结点 `i` 的左孩子编号为 `2i+1`。
- 结点 `i` 的右孩子编号为 `2i+2`。
- 结点 `i` 的双亲编号为 `(i-1)/2`。

## 链式存储

二叉树最常用二叉链表：

```c
typedef struct BiTNode {
    ElemType data;
    struct BiTNode *lchild;
    struct BiTNode *rchild;
} BiTNode, *BiTree;
```

含 `n` 个结点的二叉链表有 `2n` 个孩子指针域。由于二叉树中实际边数为 `n-1`，所以空指针域个数为：

$$
2n-(n-1)=n+1
$$

这个结论在线索二叉树中很重要：线索化正是利用这些空指针域保存前驱、后继信息。

## 三叉链表

若需要频繁访问双亲结点，可增加 `parent` 指针：

```c
typedef struct TriTNode {
    ElemType data;
    struct TriTNode *lchild;
    struct TriTNode *rchild;
    struct TriTNode *parent;
} TriTNode;
```

二叉链表从某结点找孩子很方便，但找双亲只能从根开始遍历；三叉链表用额外空间换取向上访问能力。
