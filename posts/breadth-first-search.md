---
title: Breadth First Search
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 17:36:17
katex: true
---

# 广度优先搜索

广度优先搜索（Breadth-First Search, BFS）是图的遍历算法。它从某个起始顶点出发，先访问距离起点最近的一层顶点，再访问下一层顶点。实现上的核心工具是**队列**：先被发现的顶点先扩展。

相关卡片：[[graph-basic-operations|图的基本操作]]、[[adjacency-matrix|邻接矩阵]]、[[adjacency-list|邻接表]]、[[graph-connectivity-and-components|连通分量]]。

![BFS 总览](../assets/graph-bfs-overview.svg)

## 从树的层序遍历到图的 BFS

树的层序遍历可以直接用队列：

1. 根结点入队。
2. 队头结点出队并访问。
3. 将该结点的孩子依次入队。
4. 重复直到队列为空。

图的 BFS 与它非常像，但图可能有回路。搜索某顶点的邻接点时，可能遇到已经访问过的顶点，所以 BFS 必须额外维护 `visited[]` 数组。

> [!important] 图比树多出的关键处理
> 对一个新邻接点，必须在它入队时就标记 `visited[w] = true`。否则同一个顶点可能在真正出队前，被多个已访问顶点重复发现并重复入队。

## BFS 要解决的三个小问题

| 问题          | 典型接口或结构                                                  | 说明                             |
| ----------- | -------------------------------------------------------- | ------------------------------ |
| 怎样找到某顶点的邻接点 | `FirstNeighbor(G, v)` 与 `NextNeighbor(G, v, w)`          | 不直接依赖具体存储结构，邻接矩阵和邻接表都可以实现这两个接口 |
| 怎样避免重复访问    | `visited[]`                                              | 顶点第一次被发现时置为 `true`             |
| 怎样按层扩展      | [队列](../posts/queue-definition-and-operations.md) `Queue` | 队头顶点出队并扩展，它发现的新顶点进入队尾          |

`FirstNeighbor(G, v)` 返回顶点 `v` 的第一个邻接点；若不存在，返回 `-1`。  
`NextNeighbor(G, v, w)` 在已知 `w` 是 `v` 的某个邻接点时，返回 `v` 在 `w` 之后的下一个邻接点；若 `w` 已是最后一个邻接点，返回 `-1`。

## 队列推进过程

下面的示例从顶点 `2` 出发。若邻接表中各链表的顺序固定为：

```text
1: 2, 5
2: 1, 6
3: 4, 6, 7
4: 3, 7, 8
5: 1
6: 2, 3, 7
7: 3, 4, 6, 8
8: 4, 7
```

则 BFS 序列为：

```text
2, 1, 6, 5, 3, 7, 4, 8
```

[html-card height=850 step=80](../assets/graph-bfs-process.html)

BFS 的“按层”不是靠递归深度，而是靠队列顺序自然形成：

- 起点先入队，是第 0 层。
- 起点的所有未访问邻接点依次入队，是第 1 层。
- 第 1 层顶点依次出队时，它们新发现的顶点进入队尾，是第 2 层。
- 以此类推。

## C 代码

下面用 [[graph-basic-operations#^267de5|FirstNeighbor]] 和 [[graph-basic-operations#^267de5|NextNeighbor]] 屏蔽存储结构差异。考试写伪代码时通常只要把这两个接口说明清楚即可；写 C 时可以把队列看成已经实现好的顺序队列或链队列。

```c
// Visits all vertices reachable from `start` by breadth-first search.
//
// Args:
//   graph: graph to traverse.
//   start: start vertex index.
//   visited: visited marker array shared by the outer traversal function.
//
// Side effects:
//   Calls Visit for every vertex first discovered from start and updates visited.
void BFS(const Graph *graph, int start, bool visited[]) {
    Queue queue;
    InitQueue(&queue);

    Visit(start);
    visited[start] = true;      // 入队前标记，避免被其他边重复入队。
    EnQueue(&queue, start);

    while (!QueueEmpty(&queue)) {
        int vertex;
        DeQueue(&queue, &vertex);

        // 依次检查 vertex 的所有邻接点。
        for (int neighbor = FirstNeighbor(graph, vertex);
             neighbor >= 0;
             neighbor = NextNeighbor(graph, vertex, neighbor)) {
            if (!visited[neighbor]) {
                Visit(neighbor);
                visited[neighbor] = true;
                EnQueue(&queue, neighbor);
            }
        }
    }
}
```

## 非连通图版

单次 `BFS(G, start)` 只能访问从 `start` 可达的那些顶点。若图是非连通无向图，只从一个顶点出发就无法遍历完整个图。

非连通图版写法是在外层扫描所有顶点：遇到尚未访问的顶点，就以它为起点再调用一次 BFS。

```c
// Traverses the whole graph by BFS.
//
// Args:
//   graph: graph to traverse.
//   vertexCount: number of vertices in graph.
//
// Side effects:
//   Calls BFS once for each still-unvisited search region.
void BFSTraverse(const Graph *graph, int vertexCount) {
    bool visited[MAX_VERTEX_NUM];

    for (int i = 0; i < vertexCount; ++i) {
        visited[i] = false;
    }

    for (int i = 0; i < vertexCount; ++i) {
        if (!visited[i]) {
            BFS(graph, i, visited);
        }
    }
}
```

[html-card height=820 step=80](../assets/graph-bfs-forest.html)

对无向图，有一个非常常考的结论：

> [!tip] 无向图 BFS 调用次数
> 对无向图执行完整 BFS 遍历时，`BFS` 函数的调用次数等于图的连通分量数。  
> 若无向图连通，只需调用一次；若有 $k$ 个连通分量，就需要调用 $k$ 次。

**有向图不能直接套这个结论**。因为从某顶点沿出边能否到达其他顶点，取决于弧的方向；即使底层“看起来连在一起”，也可能无法从某个起点遍历到所有顶点。


> [!important] 有向图中调用次数要具体分析
> 设有向图的弧为：`1->5`，`2->1`，`3->6`，`4->3`，`4->7`，`6->2`，`7->3`，`7->6`，`7->8`，`8->4`。  
> 若完整遍历先从 `1` 调用 BFS，只能访问 `1, 5`；之后还要从 `2`、`3`、`4` 这些尚未访问顶点继续启动 BFS，才能覆盖全图。  
> 若从 `7` 调用 BFS，则可沿 `7->3->6->2->1->5` 和 `7->8->4` 覆盖所有顶点，因此只需调用一次。  
> 所以有向图的调用次数不能只看“弱连通外形”，要看从起点沿出边实际能到达哪些顶点。

若有向图是强连通图，则从任一顶点出发都能一次 BFS 访问全部顶点。

## BFS 遍历序列

同一个图，从不同起点出发，BFS 遍历序列通常不同。例如在上面的无向图中：

- 从顶点 `1` 出发，可得到 `1, 2, 5, 6, 3, 7, 4, 8`。
- 从顶点 `2` 出发，可得到 `2, 1, 6, 5, 3, 7, 4, 8`。
- 从顶点 `3` 出发，可得到 `3, 4, 6, 7, 8, 2, 1, 5`。

即使起点相同，BFS 序列也可能受存储结构影响：

- [[adjacency-matrix|邻接矩阵]]在顶点编号固定时表示唯一，扫描邻接点通常按编号顺序进行，因此遍历序列唯一。
- [[adjacency-list|邻接表]]中同一顶点的邻接点链表顺序不唯一，因此遍历序列可能不唯一。

> [!example] 邻接表顺序改变的影响
> 若顶点 `6` 的邻接表写作 `2, 3, 7`，从 `2` 出发时会先发现 `3` 再发现 `7`。  
> 若顶点 `6` 的邻接表写作 `2, 7, 3`，则 `7` 会先于 `3` 入队，后续 BFS 序列和生成树都可能改变。

## 广度优先生成树与生成森林

在 BFS 过程中，一个顶点第一次被发现时，若是通过边 $(u,v)$ 从 `u` 发现 `v`，就把这条边记录为生成树边。所有这样的发现边构成**广度优先生成树**。

广度优先生成树的特点：

- 树根是 BFS 的起始顶点。
- 树中第 $i$ 层顶点对应图中距离起点为 $i$ 的顶点。
- 只有“第一次发现新顶点”的边进入生成树。
- 图中其余边仍存在，但不是生成树边。

由于广度优先生成树由 BFS 过程确定，所以它也受邻接点访问顺序影响：

- 基于邻接矩阵时，若顶点编号和扫描规则固定，BFS 序列与 BFS 生成树通常唯一。
- 基于邻接表时，邻接表顺序不唯一，BFS 序列与 BFS 生成树也可能不唯一。

对非连通图执行完整 BFS 遍历时，每个连通分量会得到一棵 BFS 生成树，所有这些树合起来称为**广度优先生成森林**。

## 复杂度

BFS 的辅助空间主要来自队列和 `visited[]`：

$$
O(\lvert V\rvert)
$$

时间复杂度取决于存储结构：

| 存储结构 | 顶点访问 | 邻接点查找 | 总时间复杂度 |
|---|---:|---:|---:|
| 邻接矩阵 | $O(\lvert V\rvert)$ | 每个顶点都要扫描一整行，合计 $O(\lvert V\rvert^2)$ | $O(\lvert V\rvert^2)$ |
| 邻接表 | $O(\lvert V\rvert)$ | 每条边或弧至多被检查有限次，合计 $O(\lvert E\rvert)$ | $O(\lvert V\rvert+\lvert E\rvert)$ |

> [!info] 复杂度不要只看一次 `BFS`
> 对非连通图，完整遍历会多次调用 `BFS`。但所有调用加起来，每个顶点只会被访问一次；用邻接表时，所有边或弧也只会被扫描有限次，因此总复杂度仍是 $O(\lvert V\rvert+\lvert E\rvert)$。

## 考试速记

- BFS 使用队列，DFS 使用递归或栈。
- BFS 适合求无权图从起点到其他顶点的最短路径，这一点后续最短路径部分会继续使用。
- 访问新顶点时立即标记 `visited`，再入队。
- 单次 BFS 只能覆盖从起点可达的部分。
- 无向图完整 BFS 的调用次数等于连通分量数。
- BFS 生成树记录的是“第一次发现顶点”的边。
