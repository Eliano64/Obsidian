---
title: Balanced Binary Tree Concept
tags:
  - DataStructureAndAlgorithm
  - Tree
categories:
  - Data Structure & Algorithm
date: 2026-06-27 21:38:36
katex: true
---

# 平衡二叉树概念

平衡二叉树要求树上任一结点的左子树和右子树高度差不超过 `1`。

![1000](../assets/balanced-binary-tree-concept.svg)

## 定义要点

- 限制对象是“任一结点”，不是只检查根结点。
- 比较的是左子树高度和右子树高度。
- 高度差的绝对值不能超过 `1`。

若用平衡因子表示：

$$
BF = h_{left} - h_{right}
$$

则平衡二叉树中任一结点的平衡因子只能是 `-1`、`0` 或 `1`。

## 为什么需要平衡

普通[[binary-search-tree-concept|二叉排序树]]在插入顺序不利时可能退化成接近链表的形态，查找路径变长。

平衡二叉树通过限制左右子树高度差，使树高保持较低，从而提高搜索效率。后续学习 AVL 树时，插入和删除都要在保持二叉排序树性质的同时恢复平衡。
