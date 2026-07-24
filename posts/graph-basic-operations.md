---
title: Graph Basic Operations
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 17:08:30
katex: true
---

# 图的基本操作

图的基本操作要分清两个层次：接口语义和存储结构代价。接口语义说明“这个操作要做什么”，存储结构决定“做这件事要扫描矩阵、扫描链表，还是直接改指针”。

相关卡片：[[adjacency-matrix|邻接矩阵]]、[[adjacency-list|邻接表]]、[[orthogonal-list-and-adjacency-multilist|十字链表与邻接多重表]]。

## 操作语义

| 操作                           | 含义                                                                      | 常见返回或效果                  |
| ---------------------------- | ----------------------------------------------------------------------- | ------------------------ |
| `Adjacent(G, x, y)`          | 判断图 $G$ 是否存在边 $(x,y)$ 或弧 $\langle x,y\rangle$                           | 存在返回 `true`，否则返回 `false` |
| `Neighbors(G, x)`            | 列出与顶点 $x$ 邻接的边或弧                                                        | 返回邻接点或邻接边集合              |
| `InsertVertex(G, x)`         | 在图 $G$ 中插入顶点 $x$                                                        | 新顶点初始没有关联边               |
| `DeleteVertex(G, x)`         | 从图 $G$ 中删除顶点 $x$                                                        | 同时删除与 $x$ 相关的边或弧         |
| `AddEdge(G, x, y)`           | 若边或弧不存在，则添加 $(x,y)$ 或 $\langle x,y\rangle$                              | 修改边集                     |
| `RemoveEdge(G, x, y)`        | 若边或弧存在，则删除 $(x,y)$ 或 $\langle x,y\rangle$                               | 修改边集                     |
| `FirstNeighbor(G, x)`        | 求顶点 $x$ 的第一个邻接点（有向图的情况约定寻找$x$的第一条出边的弧头）                                 | 存在则返回顶点号，否则返回 `-1`       |
| `NextNeighbor(G, x, y)`      | 已知 $y$ 是 $x$ 的邻接点，求 $x$ 除 $y$ 外的下一个邻接点（有向图的情况约定$x\to y$,寻找$x$的下一条出边的弧头） | 若 $y$ 是最后一个邻接点，则返回 `-1`  |
| `Get_edge_value(G, x, y)`    | 获取边或弧对应的权值                                                              | 返回权值                     |
| `Set_edge_value(G, x, y, v)` | 设置边或弧对应的权值为 $v$                                                         | 修改权值                     |

^267de5

## 邻接矩阵与邻接表复杂度

| 操作                           | 邻接矩阵                                                           | 邻接表                                                                                         |
| ---------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `Adjacent(G, x, y)`          | $O(1)$，直接访问 `edge[x][y]`                                       | $O(1)\sim O(\lvert V\rvert)$，扫描 $x$ 的链表                                                     |
| `Neighbors(G, x)`，无向图        | $O(\lvert V\rvert)$，扫描一行或一列                                    | $O(1)\sim O(\lvert V\rvert)$，遍历 $x$ 的链表                                                     |
| `Neighbors(G, x)`，有向图        | $O(\lvert V\rvert)$，扫描第 $x$ 行；再加上$O(\lvert V\rvert)$，扫描第 $x$ 列 | $O(1)\sim O(\lvert V\rvert)$，遍历 $x$ 的链表；再加上$O(\lvert E \rvert + \lvert V\rvert)$，查找$x$所有入边； |
| `InsertVertex(G, x)`         | $O(1)$，使用预留矩阵空间时直接增加顶点计数                                       | $O(1)$，新增顶点表结点并令链表为空                                                                        |
| `DeleteVertex(G, x)`，无向图     | $O(\lvert V\rvert)$，清空对应行列或移动数据                                | $O(\lvert V\rvert+2\lvert E\rvert)$，删除顶点并清理所有相关边结点                                          |
| `DeleteVertex(G, x)`，有向图     | $O(\lvert V\rvert)$，清空对应行列或移动数据                                | 删出边加删入边，按完整邻接表扫描计为 $O(\lvert V\rvert+\lvert E\rvert)$                                       |
| `AddEdge(G, x, y)`           | $O(1)$，写矩阵单元                                                   | $O(1)$，若已确认不存在，可头插或尾插边结点                                                                    |
| `RemoveEdge(G, x, y)`        | $O(1)$，清矩阵单元                                                   | $O(1)\sim O(\lvert V\rvert)$，在$x$对应链表中查找并删除                                                 |
| `FirstNeighbor(G, x)`，       | $O(1)\sim O(\lvert V\rvert)$，从行首扫描到第一个非零单元                     | $O(1)$，返回链表首结点                                                                              |
| `NextNeighbor(G, x, y)`      | $O(1)\sim O(\lvert V\rvert)$，从 $y$ 后继续扫描                       | $O(1)$，若已持有 $y$ 所在边结点，取 `next`                                                              |
| `Get_edge_value(G, x, y)`    | $O(1)$                                                         | $O(1)\sim O(\lvert V\rvert)$，核心是先找到边                                                        |
| `Set_edge_value(G, x, y, v)` | $O(1)$                                                         | $O(1)\sim O(\lvert V\rvert)$，核心是先找到边                                                        |

> [!warning] 复杂度依赖操作前提
> 表中 `AddEdge` 的邻接表复杂度按“已经确认边不存在”计算；若插入前还要调用 `Adjacent` 查重，则需要加上查找成本。
> `NextNeighbor` 的邻接表 $O(1)$ 也默认已经定位到 $y$ 所在的边结点；如果只给顶点编号而没有边结点位置，仍需要先扫描链表。
> “扫描整个邻接表”按顶点表和所有边结点一起计算，写作 $O(\lvert V\rvert+\lvert E\rvert)$。
> 无向图邻接表删除一个顶点时，既要处理顶点表，又要清理相关边结点；教材常写作 $O(\lvert V\rvert+2\lvert E\rvert)$。

## 存储结构总表

| 存储结构 | 空间复杂度 | 找相邻边 | 删除边或顶点 | 适用对象 | 表示方式 |
|---|---|---|---|---|---|
| [[adjacency-matrix]] | $O(\lvert V\rvert^2)$ | 遍历对应行或列，判断指定边为 $O(1)$ | 删除边方便；删除顶点可能需要移动大量数据 | 稠密图 | 唯一 |
| [[adjacency-list]] | 无向图 $O(\lvert V\rvert+2\lvert E\rvert)$；有向图 $O(\lvert V\rvert+\lvert E\rvert)$ | 有向图找入边要扫描整个邻接表，其余较方便 | 无向图中删除边或顶点不方便 | 稀疏图 | 不唯一 |
| [[orthogonal-list-and-adjacency-multilist]] | $O(\lvert V\rvert+\lvert E\rvert)$ | 入边、出边都方便 | 较方便 | 有向图 | 不唯一 |
| [[orthogonal-list-and-adjacency-multilist]] | $O(\lvert V\rvert+\lvert E\rvert)$ | 方便 | 较方便 | 无向图 | 不唯一 |
