---
title: Undirected And Directed Graph
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 10:46:30
katex: true
---

# 无向图与有向图

图的边是否有方向，决定它是无向图还是有向图。

![1000](../assets/graph-directed-undirected.svg)

## 无向图

若 `E` 是无向边的有限集合，则图 `G` 为无向图。

无向边是顶点的无序对，记为：

$$
(v,w)
$$

因为没有方向，所以：

$$
(v,w)=(w,v)
$$

相关术语：

- 顶点 `v` 和 `w` 互为**邻接点**。
- 边 `(v,w)` **依附于**顶点 `v` 和 `w`。
- 也可以说边 `(v,w)` 与顶点 `v`、`w` **相关联**。

例如：

$$
V=\{A,B,C,D,E\}
$$

$$
E=\{(A,B),(B,D),(B,E),(C,D),(C,E),(D,E)\}
$$

## 有向图

若 `E` 是有向边的有限集合，则图 `G` 为有向图。有向边$v\to w$也称为**弧**。

弧是顶点的有序对，记为：

$$
\langle v,w\rangle
$$

其中：

- `v` 称为**弧尾**。
- `w` 称为**弧头**。
- $\langle v,w\rangle$ 表示从 `v` 到 `w` 的弧。
- 也称 `v` **邻接到** `w`，或 `w` **邻接自** `v`。

有向边有方向，因此：

$$
\langle v,w\rangle \ne \langle w,v\rangle
$$

例如：

$$
V=\{A,B,C,D,E\}
$$

$$
E=\{\langle A,B\rangle,\langle A,C\rangle,\langle A,D\rangle,\langle A,E\rangle,\langle B,A\rangle,\langle B,C\rangle,\langle B,E\rangle,\langle C,D\rangle\}
$$

## 查阅重点

无向图中，`(v,w)` 和 `(w,v)` 是同一条边；有向图中，$\langle v,w\rangle$ 和 $\langle w,v\rangle$ 是两条方向相反的弧。

这一区别会影响[[graph-degree|度、入度与出度]]、[[complete-graph|完全图]]、[[weighted-graph|带权图]]、|图的存储结构]]和图的遍历。
