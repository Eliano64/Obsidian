---
title: Graph Relation Concepts Table
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 11:31:30
katex: true
---


| 概念                                                  | 适用对象           | 一句话定义                                                     |
| --------------------------------------------------- | -------------- | --------------------------------------------------------- |
| [度 $TD(v)$](Graph-Degree.md)                        | 无向图、有向图        | 与顶点 $v$ 相关的边或弧的总数                                         |
| [入度 $ID(v)$](Graph-Degree.md)                       | 有向图            | 以 $v$ 为终点的弧数                                              |
| [出度 $OD(v)$](Graph-Degree.md)                       | 有向图            | 以 $v$ 为起点的弧数                                              |
| [路径](Graph-Path-And-Distance.md)                    | 无向图、有向图        | 从一个顶点到另一个顶点的顶点序列                                          |
| [回路，也称环](Graph-Path-And-Distance.md)                | 无向图、有向图        | 起点和终点相同的路径                                                |
| [简单路径](Graph-Path-And-Distance.md)                  | 无向图、有向图        | 顶点不重复出现的路径                                                |
| [简单回路](Graph-Path-And-Distance.md)                  | 无向图、有向图        | 除首尾同一顶点外，其余顶点不重复的回路                                       |
| [路径长度](Graph-Path-And-Distance.md)                  | 无向图、有向图        | 路径上边或弧的数目                                                 |
| [距离](Graph-Path-And-Distance.md)                    | 无向图、有向图        | 两点间最短路径的长度                                                |
| [连通](Graph-Connectivity-And-Components.md)          | 无向图            | 两个顶点之间存在路径                                                |
| [强连通](Graph-Connectivity-And-Components.md)         | 有向图            | 两个顶点互相都有路径可达                                              |
| [连通图](Graph-Connectivity-And-Components.md)         | 无向图            | 任意两个顶点都连通                                                 |
| [非连通图](Graph-Connectivity-And-Components.md)        | 无向图            | 至少有一对顶点不连通                                                |
| [强连通图](Graph-Connectivity-And-Components.md)        | 有向图            | 任意两个顶点都强连通                                                |
| [子图](Graph-Subgraph.md)                             | 无向图、有向图        | 从原图中取部分顶点和部分边或弧                                           |
| [生成子图](Graph-Subgraph.md)                           | 无向图、有向图        | 保留原图全部顶点的子图                                               |
| [连通子图](Graph-Subgraph.md)                           | 无向图            | 本身连通的子图                                                   |
| [连通分量](Graph-Connectivity-And-Components.md)        | 无向图            | 极大连通子图                                                    |
| [强连通分量](Graph-Connectivity-And-Components.md)       | 有向图            | 极大强连通子图                                                   |
| [生成树](Spanning-Tree.md)                             | 连通无向图          | 包含全部顶点的极小连通子图                                             |
| [生成森林](Spanning-Tree.md)                            | 非连通无向图         | 各连通分量的生成树组成的森林                                            |
| [最小生成树](Minimum-Spanning-Tree.md)                   | 带权无向连通图        | 权值总和最小的生成树                                                |
| [Kruskal](Minimum-Spanning-Tree.md)                 | 带权无向图          | 按边权从小到大选不成环的边                                             |
| [Prim](Minimum-Spanning-Tree.md)                    | 带权无向图          | 从树集合出发，每次接入最近的外部顶点                                        |
| [边的权](Weighted-Graph.md)                            | 无向图、有向图        | 边或弧上标注的数值                                                 |
| [带权图，也称网](Weighted-Graph.md)                        | 无向图、有向图        | 边或弧带有权值的图                                                 |
| [带权路径长度](Weighted-Graph.md)                         | 带权图            | 路径上所有边或弧的权值之和                                             |
| [单源最短路径](Shortest-Path-Algorithms.md)               | 无向图、有向图、带权图    | 从一个源点到其他各顶点的最短路径                                          |
| [每对顶点间最短路径](Shortest-Path-Algorithms.md)            | 有向图、无向图、带权图    | 求任意两个顶点之间的最短路径                                            |
| [负权回路](Shortest-Path-Algorithms.md)                 | 带权图            | 回路上边权之和为负                                                 |
| [邻接矩阵](Adjacency-Matrix.md)                         | 无向图、有向图、带权图    | 用 $\lvert V\rvert\times \lvert V\rvert$ 矩阵表示顶点之间是否有边、弧或权值 |
| [无向图邻接矩阵](Adjacency-Matrix.md)                      | 无向图            | $A[i][j]=A[j][i]$，矩阵关于主对角线对称                              |
| [有向图邻接矩阵](Adjacency-Matrix.md)                      | 有向图            | $A[i][j]$ 表示从 $v_i$ 到 $v_j$ 的弧                            |
| [邻接表](Adjacency-List.md)                            | 无向图、有向图、带权图    | 顶点表顺序存储顶点，每个顶点后接链表保存邻接点                                   |
| [无向图邻接表](Adjacency-List.md)                         | 无向图            | 每条边在两个端点的链表中各存一次                                          |
| [有向图邻接表](Adjacency-List.md)                         | 有向图            | 每条弧只存到起点的链表中，默认保存出边                                       |
| [十字链表](Orthogonal-List-And-Adjacency-Multilist.md)  | 有向图            | 一个弧结点同时挂入弧尾的出边链和弧头的入边链                                    |
| [邻接多重表](Orthogonal-List-And-Adjacency-Multilist.md) | 无向图            | 一条无向边只存一个边结点，同时挂入两个端点的边链                                  |
| [广度优先搜索](Graph-Traversal.md)                        | 无向图、有向图        | 借助队列从起点按层扩展访问顶点                                           |
| [广度优先生成树](Graph-Traversal.md)                       | 连通图的 BFS 过程    | 由 BFS 第一次发现新顶点时经过的边构成                                     |
| [广度优先生成森林](Graph-Traversal.md)                      | 非连通图的完整 BFS 过程 | 各连通分量的 BFS 生成树组成的森林                                       |
| [深度优先搜索](Graph-Traversal.md)                        | 无向图、有向图        | 沿未访问邻接点不断深入，走不动再回退                                        |
| [深度优先生成树](Graph-Traversal.md)                       | 连通图的 DFS 过程    | 由 DFS 第一次递归进入新顶点时经过的边构成                                   |
| [深度优先生成森林](Graph-Traversal.md)                      | 非连通图的完整 DFS 过程 | 各连通分量的 DFS 生成树组成的森林                                       |
| [DAG](DAG-And-AOV-Network.md)                       | 有向图            | 不存在有向环的有向图                                                |
| [AOV 网](DAG-And-AOV-Network.md)                     | 有向无环图          | 顶点表示活动，边表示活动先后约束                                          |
| [拓扑排序](Topological-Sorting.md)                      | DAG、AOV 网      | 使每条路径起点都排在终点前面的顶点序列                                       |
| [逆拓扑排序](Topological-Sorting.md)                     | DAG、AOV 网      | 从后继约束末端开始输出的反向拓扑序列                                        |
| [AOE 网](AOE-Network-And-Critical-Path.md)           | 带权有向无环图        | 顶点表示事件，边表示活动，边权表示活动耗时                                     |
| [关键路径](AOE-Network-And-Critical-Path.md)            | AOE 网          | 从源点到汇点的最长路径                                               |
| [关键活动](AOE-Network-And-Critical-Path.md)            | AOE 网          | 时间余量为 0 的活动                                               |
| [稀疏图](Sparse-And-Dense-Graph.md)                    | 无向图、有向图        | 边数较少的图                                                    |
| [稠密图](Sparse-And-Dense-Graph.md)                    | 无向图、有向图        | 边数较多的图                                                    |
| [树](Tree-As-Graph-Special-Form.md)                  | 无向图            | 连通且无回路的无向图                                                |
| [森林](Tree-As-Graph-Special-Form.md)                 | 无向图            | 若干棵互不相交的树组成                                               |
| [有向树](Tree-As-Graph-Special-Form.md)                | 有向图            | 一个顶点入度为 0，其余顶点入度均为 1                                      |
| [无向完全图](Complete-Graph.md)                          | 无向图            | 任意两个不同顶点之间都有边                                             |
| [有向完全图](Complete-Graph.md)                          | 有向图            | 任意两个不同顶点之间都有方向相反的两条弧                                      |

