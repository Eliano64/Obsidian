---
title: Graph Definition 
tags:
  - DataStructureAndAlgorithm
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 10:46:30 
katex: true
---

# 图的定义

图由顶点集和边集组成，记为：

$$
G=(V,E)
$$

- `G`：graph，表示图。
- `V`：vertex set，表示顶点集。
- `E`：edge set，表示边集。
- `V(G)`：图 `G` 的顶点集，是有限**非空**集。
- `E(G)`：图 `G` 中顶点之间关系的集合，可以为空。

若：

$$
V=\{v_1,v_2,\ldots,v_n\}
$$

则：

- $|V|$ 表示图中顶点个数，也称图的阶。
- $|E|$ 表示图中边的条数。

![1000](../assets/graph-definition.svg)

## **图不能为空**

线性表可以是空表，树可以是空树，但图不可以没有顶点。

图允许：

- `V` 非空，`E` 非空。
- `V` 非空，`E = ∅`。

图不允许：

- `V = ∅`。

因此，只有顶点而没有边的结构仍然是图；没有顶点的结构不是图。

## 图逻辑结构的应用

图适合表示“多对多”的关系：

| 顶点 `V` | 边 `E` | 关系含义 |
|---|---|---|
| 车站 | 铁路 | 车站之间是否有铁路连接 |
| 路口 | 道路 | 路口之间是否有道路连接 |
| 用户 | 好友关系 | 两个用户是否互为好友 |
| 用户 | 关注关系 | 一个用户是否关注另一个用户 |

若关系没有方向，通常抽象为[[undirected-and-directed-graph#无向图|无向图]]；若关系有方向，通常抽象为[[undirected-and-directed-graph#有向图|有向图]]。
