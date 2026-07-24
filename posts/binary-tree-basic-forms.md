---
title: Binary Tree Basic Forms
tags:
  - Tree
categories:
  - Data Structure & Algorithm
date: 2026-06-27 21:38:36
katex: true
---

# 二叉树的五种基本形态

二叉树允许为空，并且左右子树的位置有含义。因此，从根结点角度看，二叉树有五种基本形态。

![1000](../assets/binary-tree-basic-forms.svg)

## 五种形态

| 形态 | 说明 |
| --- | --- |
| 空二叉树 | 没有任何结点，是递归定义和算法递归结束的边界。 |
| 只有根结点 | 根结点没有左子树，也没有右子树。 |
| 只有左子树 | 根结点有左子树，右子树为空。 |
| 只有右子树 | 根结点有右子树，左子树为空。 |
| 左右子树都有 | 根结点同时有左子树和右子树。 |

## 查阅重点

只含一个孩子时，不要只说“有一个孩子”，要明确是左孩子还是右孩子。这个位置差异会影响：

- [[binary-tree-traversal|遍历序列]]
- [[binary-tree-storage|顺序存储中的空位]]
- [[complete-binary-tree|完全二叉树]]的判断
- [[threaded-binary-tree|线索二叉树]]中前驱、后继线索的含义
