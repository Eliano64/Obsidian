---
title: Depth First Search
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 17:54:13
katex: true
---

# 深度优先搜索

深度优先搜索（Depth-First Search, DFS）是图的遍历算法。它从某个起始顶点出发，沿着一个未访问邻接点不断深入；当前顶点没有未访问邻接点时，再返回上一层，继续尝试上一层的其他邻接点。


![DFS 总览](../assets/graph-dfs-overview.svg)

# 从树的先根遍历到图的 DFS

树的深度优先遍历通常对应先根遍历：

1. 访问当前结点。
2. 递归访问第一个孩子及其子树。
3. 第一个孩子子树处理完后，再处理下一个孩子子树。
4. 所有孩子处理完，返回上一层。

图的 DFS 与它类似，但图可能有回路。树中从父结点走向孩子时，孩子一定没访问过；图中搜索邻接点时，可能遇到已经访问过的顶点，所以必须维护 `visited[]`。

> [!important] 
> `visited[v] = true` 表示顶点 `v` 已经被发现。递归版本 DFS 的“当前搜索路径”隐含保存在函数调用栈中，这一点也是有向图判环的基础。

# DFS 算法

## 递归调用栈过程

若邻接表顺序为：

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

从顶点 `2` 出发，DFS 序列为：

```text
2, 1, 5, 6, 3, 4, 7, 8
```

[html-card height=850 step=80](../assets/graph-dfs-process.html)

DFS 的“深度优先”体现在：只要当前顶点还能找到未访问邻接点，就马上递归进入该邻接点，而不是先把当前顶点的所有邻接点都收集起来。

## C 代码

下面仍使用 `FirstNeighbor` 和 `NextNeighbor` 屏蔽邻接矩阵、邻接表等存储结构差异。

```c
// Visits all vertices reachable from `vertex` by depth-first search.
//
// Args:
//   graph: graph to traverse.
//   vertex: current vertex index.
//   visited: visited marker array shared by the outer traversal function.
//
// Side effects:
//   Calls Visit when a vertex is first discovered and updates visited.
void DFS(const Graph *graph, int vertex, bool visited[]) {
    Visit(vertex);
    visited[vertex] = true;

    // 从 vertex 的第一个邻接点开始，逐个尝试。
    for (int neighbor = FirstNeighbor(graph, vertex);
         neighbor >= 0;
         neighbor = NextNeighbor(graph, vertex, neighbor)) {
        if (!visited[neighbor]) {
            // 一旦发现未访问邻接点，立即深入递归。
            DFS(graph, neighbor, visited);
        }
    }
}
```

## 非连通图版

单次 `DFS(G, start)` 只能访问从 `start` 可达的顶点。若图非连通，必须用外层循环扫描所有顶点；遇到尚未访问的顶点，就从它重新启动 DFS。

```c
// Traverses the whole graph by DFS.
//
// Args:
//   graph: graph to traverse.
//   vertexCount: number of vertices in graph.
//
// Side effects:
//   Calls DFS once for each still-unvisited search region.
void DFSTraverse(const Graph *graph, int vertexCount) {
    bool visited[MAX_VERTEX_NUM];

    for (int i = 0; i < vertexCount; ++i) {
        visited[i] = false;
    }

    for (int i = 0; i < vertexCount; ++i) {
        if (!visited[i]) {
            DFS(graph, i, visited);
        }
    }
}
```

对无向图，完整 DFS 遍历时 `DFS` 函数的调用次数等于连通分量数。连通图只需调用一次；有 $k$ 个连通分量，就需要调用 $k$ 次。

有向图不能只看弱连通外形。若起始顶点沿弧方向能到达其他所有顶点，则只需调用一次；若有向图是强连通图，则从任一顶点出发都只需调用一次。

# DFS 遍历序列

同一个图，从不同起点出发，DFS 遍历序列通常不同。例如在前面的无向图中：

- 从顶点 `2` 出发，可得到 `2, 1, 5, 6, 3, 4, 7, 8`。
- 从顶点 `3` 出发，可得到 `3, 4, 7, 6, 2, 1, 5, 8`。
- 从顶点 `1` 出发，可得到 `1, 2, 6, 3, 4, 7, 8, 5`。

即使起点相同，DFS 序列也可能受存储结构影响：

- [[Adjacency-Matrix|邻接矩阵]]在顶点编号固定时表示唯一，通常按编号顺序扫描邻接点，因此 DFS 序列唯一。
- [[Adjacency-List|邻接表]]中同一顶点的邻接点链表顺序不唯一，因此 DFS 序列可能不唯一。

> [!example] 邻接表顺序改变的影响
> 若顶点 `2` 的邻接表写作 `1, 6`，从 `2` 出发会先深入 `1`。  
> 若顶点 `2` 的邻接表写作 `6, 1`，从 `2` 出发会先深入 `6`，DFS 序列可能变为 `2, 6, 7, 8, 4, 3, 1, 5`。

# 深度优先生成树与生成森林

DFS 过程中，一个顶点第一次被发现时，若是通过边 $(u,v)$ 从 `u` 进入 `v`，就把这条边记录为生成树边。所有这样的边构成**深度优先生成树**。

深度优先生成树的特点：

- 树根是 DFS 的起始顶点。
- 每条树边对应一次递归进入新顶点。
- 图中被跳过的已访问邻接边不会进入生成树。
- 基于邻接矩阵时，若顶点编号和扫描规则固定，DFS 序列与 DFS 生成树通常唯一。
- 基于邻接表时，邻接表顺序不唯一，DFS 序列与 DFS 生成树也可能不唯一。

对非连通图执行完整 DFS 遍历时，每个连通分量会得到一棵 DFS 生成树，所有这些树合起来称为**深度优先生成森林**。

# 复杂度

递归 DFS 的辅助空间来自函数调用栈和 `visited[]`。最坏情况下，递归深度可达 $\lvert V\rvert$，所以空间复杂度为：

$$
O(\lvert V\rvert)
$$

若某些图形结构使递归深度很浅，调用栈本身可能接近 $O(1)$；但一般考试分析 DFS 时按最坏情况写 $O(\lvert V\rvert)$。

时间复杂度取决于存储结构：

| 存储结构 |                顶点访问 |                                邻接点查找 |                             总时间复杂度 |
| ---- | ------------------: | -----------------------------------: | ---------------------------------: |
| 邻接矩阵 | $O(\lvert V\rvert)$ | 每个顶点都要扫描一整行，合计 $O(\lvert V\rvert^2)$ |              $O(\lvert V\rvert^2)$ |
| 邻接表  | $O(\lvert V\rvert)$ |   所有边或弧被扫描有限次，合计 $O(\lvert E\rvert)$ | $O(\lvert V\rvert+\lvert E\rvert)$ |

# DFS 判断图是否有环

DFS 第一次发现一个顶点时，会把发现它的边记为 DFS 树边。

[html-card height=650 step=80](../assets/graph-dfs-directed-cycle.html)

## 无向图

记录每个顶点在 BFS 树中的父顶点 `parent`。

从 `u` 检查邻接点 `v` 时：

- `v` 未访问：令 `parent[v] = u`，递归进入 `v`。
- `v` 已访问且 `v == parent[u]`：若 DFS 从 `u` 发现 `v`，后来处理 `v` 时必然会再次看到已经访问的 `u`。这是 `u` 的来路，忽略。
- `v` 已访问且 `v != parent[u]`：除了 `u` 以外， `v` 还与另一点相连，说明存在从源点出发的多条路径能到达 `v` ，由于是无向图，说明至少存在一条从源点到源点的路径，即存在回路。

## 有向图

即使存在从源点出发的多条路径能到达 `v` ，也不能保证存在从 `v` 返回源点的有向路径。

因此对有向图判环时，我们应利用递归栈的性质，使用三种状态：

| 状态 | 含义 |
|---|---|
| `UNVISITED` | 尚未进入 DFS |
| `VISITING` | 已进入 DFS，仍在当前递归路径上 |
| `DONE` | 所有出边均已处理，已经退出当前路径 |

在 `u` 对应的递归函数 $dfs(u)$ 中将要进入能到达的 `v` 对应的 $dfs(v)$ 之前检查：

- `v` 是 `UNVISITED`：说明 `u` 与 `v` 此时不存在环，递归进入 `v` 。
- `v` 是 `VISITING`：当前路径中已有 $v\to\cdots\to u$，再加上 $u\to v$，形成有向环。
- `v` 是 `DONE`：`v` 已退出当前路径，不能由 $u\to v$ 推出 $v$ 能返回 `u`，不能据此判环。