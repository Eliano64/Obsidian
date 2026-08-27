---
title: Graph Traversal
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 17:36:17
katex: true
---
# 两种搜索算法

图的遍历：从一个顶点出发，沿边逐步发现所有可达顶点，并保证每个顶点只处理一次。

深度优先搜索和广度优先搜索是图的两种遍历方式。
# 深度优先搜索

深度优先搜索（Depth-First Search, DFS）是图的遍历算法。它从某个起始顶点出发，沿着一个未访问邻接点不断深入；当前顶点没有未访问邻接点时，再返回上一层，继续尝试上一层的其他邻接点。


![DFS 总览](../assets/graph-dfs-overview.svg)

## 从树的先根遍历到图的 DFS

树的深度优先遍历通常对应先根遍历：

1. 访问当前结点。
2. 递归访问第一个孩子及其子树。
3. 第一个孩子子树处理完后，再处理下一个孩子子树。
4. 所有孩子处理完，返回上一层。

图的 DFS 与它类似，但图可能有回路。树中从父结点走向孩子时，孩子一定没访问过；图中搜索邻接点时，可能遇到已经访问过的顶点，所以必须维护 `visited[]`。

> [!important] 
> `visited[v] = true` 表示顶点 `v` 已经被发现。递归版本 DFS 的“当前搜索路径”隐含保存在函数调用栈中，这一点也是有向图判环的基础。

## DFS 算法

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

下面使用 `FirstNeighbor` 和 `NextNeighbor` 屏蔽邻接矩阵、邻接表等存储结构差异。

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

## 非连通图

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

有向图则不能确定次数。例如某有向图为弱连通图，但选择的起始顶点恰好沿弧方向能到达其他所有顶点，只需调用一次。

当然，若有向图是强连通图，则从任一顶点出发都只需调用一次。

## 复杂度

递归 DFS 的辅助空间来自函数调用栈和 `visited[]`。最坏情况下，递归深度可达 $\lvert V\rvert$，所以空间复杂度为：

$$
O(\lvert V\rvert)
$$

时间复杂度取决于存储结构：

| 存储结构 |                顶点访问 |                                邻接点查找 |                             总时间复杂度 |
| ---- | ------------------ | ----------------------------------- | ---------------------------------: |
| 邻接矩阵 | $O(\lvert V\rvert)$ | 每个顶点都要扫描一整行，合计 $O(\lvert V\rvert^2)$ |              $O(\lvert V\rvert^2)$ |
| 邻接表  | $O(\lvert V\rvert)$ |   所有边或弧被扫描有限次，合计 $O(\lvert E\rvert)$ | $O(\lvert V\rvert+\lvert E\rvert)$ |


## DFS 遍历序列

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

## 深度优先生成树与生成森林

DFS 过程中，一个顶点第一次被发现时，若是通过边 $(u,v)$ 从 `u` 进入 `v`，就把这条边记录为生成树边。所有这样的边构成**深度优先生成树**。

深度优先生成树的特点：

- 树根是 DFS 的起始顶点。
- 每条树边对应一次递归进入新顶点。
- 图中被跳过的已访问邻接边不会进入生成树。
- 基于邻接矩阵时，若顶点编号和扫描规则固定，DFS 序列与 DFS 生成树通常唯一。
- 基于邻接表时，邻接表顺序不唯一，DFS 序列与 DFS 生成树也可能不唯一。

对非连通图执行完整 DFS 遍历时，每个连通分量会得到一棵 DFS 生成树，所有这些树合起来称为**深度优先生成森林**。


## DFS 判断图是否有环

DFS 第一次发现一个顶点时，会把发现它的边记为 DFS 树边。

[html-card height=650 step=80](../assets/graph-dfs-directed-cycle.html)

### 无向图

记录每个顶点在 BFS 树中的父顶点 `parent`。

从 `u` 检查邻接点 `v` 时：

- `v` 未访问：令 `parent[v] = u`，递归进入 `v`。
- `v` 已访问且 `v == parent[u]`：若 DFS 从 `u` 发现 `v`，后来处理 `v` 时必然会再次看到已经访问的 `u`。这是 `u` 的来路，忽略。
- `v` 已访问且 `v != parent[u]`：除了 `u` 以外， `v` 还与另一点相连，说明存在从源点出发的多条路径能到达 `v` ，由于是无向图，说明至少存在一条从源点到源点的路径，即存在回路。

### 有向图

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

# 广度优先搜索

广度优先搜索（Breadth-First Search, BFS）是图的遍历算法。它从某个起始顶点出发，先访问距离起点最近的一层顶点，再访问下一层顶点。实现上的核心工具是**队列**：先被发现的顶点先扩展。

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

## BFS 算法

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

## 非连通图

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



对无向图执行完整 BFS 遍历时，`BFS` 函数的调用次数等于图的连通分量数。若无向图连通，只需调用一次；若有 $k$ 个连通分量，就需要调用 $k$ 次。

有向图则不能确定次数。例如某有向图为弱连通图，但选择的起始顶点恰好沿弧方向能到达其他所有顶点，只需调用一次。

当然，若有向图是强连通图，则从任一顶点出发都只需调用一次。

## BFS 遍历序列

同一个图，从不同起点出发，BFS 遍历序列通常不同。例如在上面的无向图中：

- 从顶点 `1` 出发，可得到 `1, 2, 5, 6, 3, 7, 4, 8`。
- 从顶点 `2` 出发，可得到 `2, 1, 6, 5, 3, 7, 4, 8`。
- 从顶点 `3` 出发，可得到 `3, 4, 6, 7, 8, 2, 1, 5`。

即使起点相同，BFS 序列也可能受存储结构影响：

- [[Adjacency-Matrix|邻接矩阵]]在顶点编号固定时表示唯一，扫描邻接点通常按编号顺序进行，因此遍历序列唯一。
- [[Adjacency-List|邻接表]]中同一顶点的邻接点链表顺序不唯一，因此遍历序列可能不唯一。

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

BFS 的辅助空间来自队列和 `visited[]`：

$$
O(\lvert V\rvert)
$$

时间复杂度取决于存储结构：

| 存储结构 | 顶点访问 | 邻接点查找 | 总时间复杂度 |
|---|---:|---:|---:|
| 邻接矩阵 | $O(\lvert V\rvert)$ | 每个顶点都要扫描一整行，合计 $O(\lvert V\rvert^2)$ | $O(\lvert V\rvert^2)$ |
| 邻接表 | $O(\lvert V\rvert)$ | 每条边或弧至多被检查有限次，合计 $O(\lvert E\rvert)$ | $O(\lvert V\rvert+\lvert E\rvert)$ |
## BFS 判断无向图是否有环

对无向图执行 BFS 时，除 `visited[]` 外，再记录每个顶点在 BFS 树中的父顶点 `parent`。

令 `u` 有邻接点 `v` ，当 `u` 出队时：

- `v` 未访问：令 `parent[v] = u`，标记后入队。
- `v` 已访问且 `v == parent[u]`：`v` 先于 `u` 出队，且 `v` 的出队带来 `u` 的入队。不说明存在回路。
- `v` 已访问且 `v != parent[u]`：除了 `u` 以外， `v` 还与另一点相连，说明存在从源点出发的多条路径能到达 `v` ，由于是无向图，说明至少存在一条从源点到源点的路径，即存在回路。

> [!warning] BFS 不能判断有向图是否存在环
> 
> 由上面的判断可知，`v` 已访问且 `v != parent[u]` 只能说明存在从源点出发的多条路径能到达 `v` ，但由于有向图的边存在方向，因而不能判断是否存在一条从源点到源点的路径。
>
>究其原因，还是在于 `visited[]` 和 `parent[]` 只能保存点被遍历到的先后顺序，无法得知它们之间的多步可达性。
>
>而对于 DFS ，比起 BFS 它利用了递归栈，可以很好地保存这一信息。
