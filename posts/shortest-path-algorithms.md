---
title: Shortest Path Algorithms 
tags:
  - DataStructureAndAlgorithm
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 21:30:32 
katex: true
---
最短路径问题讨论的是：在图中，从一个顶点走到另一个顶点，怎样使路径长度最小。

这里的“长度”要分清两种情况：

- **无权图**：每条边都可看成权值为 1，路径长度就是路径上的边数。
- **带权图**：路径长度是路径上各边权值之和，也叫**带权路径长度**。

## 问题分类

### 单源最短路径

给定一个源点 $s$，求 $s$ 到其他各顶点的最短路径。

常见方法：

- 无权图：BFS。
- 非负权图：Dijkstra。
- 允许负权边但无负权回路：可用 Bellman-Ford 或 SPFA。

### 每对顶点间的最短路径

求图中任意两个顶点 $V_i$ 到 $V_j$ 的最短路径。

常见方法：

- Floyd。
- 也可以对每个顶点各运行一次 Dijkstra，但前提仍然是边权非负。

## BFS：无权图的单源最短路径

无权图可以看成每条边权值都为 1 的特殊带权图。BFS 从源点按“距离层次”向外扩展：先访问距离为 1 的点，再访问距离为 2 的点。一个顶点第一次被 BFS 访问时，它的距离已经是从源点到它的最短距离。

[html-card height=820](../assets/shortest-path-bfs.html)

### 数组含义

- `visited[v]`：顶点 `v` 是否已经被访问。
- `dist[v]`：源点到 `v` 的最短边数；不可达时保持 `INF`。
- `prev[v]`：最短路径上 `v` 的前驱顶点；源点或不可达点记为 `-1`。

例如从顶点 2 出发，如果最后 `dist[8] = 3`，且 `prev[8] = 7`、`prev[7] = 6`、`prev[6] = 2`，则路径可以反向追溯为：

```text
8 <- 7 <- 6 <- 2
```

正向路径就是：

```text
2 -> 6 -> 7 -> 8
```

### C 代码

```c
#include <stdbool.h>
#include <stdio.h>

#define MAX_VERTEX_NUM 100
#define INF 0x3f3f3f3f

typedef struct {
    int vexNum;
    int edgeNum;
    int arcs[MAX_VERTEX_NUM][MAX_VERTEX_NUM];
} MGraph;

typedef struct {
    int data[MAX_VERTEX_NUM];
    int front;
    int rear;
} Queue;

static void InitQueue(Queue *queue) {
    queue->front = 0;
    queue->rear = 0;
}

static bool IsEmpty(const Queue *queue) {
    return queue->front == queue->rear;
}

static void EnQueue(Queue *queue, int vertex) {
    queue->data[queue->rear++] = vertex;
}

static int DeQueue(Queue *queue) {
    return queue->data[queue->front++];
}

/**
 * Uses BFS to compute shortest paths in an unweighted graph.
 *
 * Args:
 *   graph: Adjacency matrix. graph->arcs[u][v] != 0 means u is adjacent to v.
 *   source: The source vertex index.
 *   dist: Output array. dist[v] is the shortest edge count from source to v.
 *   prev: Output array. prev[v] is the predecessor of v on the shortest path.
 */
void BfsShortestPath(const MGraph *graph, int source, int dist[], int prev[]) {
    bool visited[MAX_VERTEX_NUM] = {false};
    Queue queue;
    InitQueue(&queue);

    for (int v = 0; v < graph->vexNum; ++v) {
        dist[v] = INF;
        prev[v] = -1;
    }

    visited[source] = true;
    dist[source] = 0;
    EnQueue(&queue, source);

    while (!IsEmpty(&queue)) {
        int current = DeQueue(&queue);

        for (int next = 0; next < graph->vexNum; ++next) {
            if (graph->arcs[current][next] == 0 || visited[next]) {
                continue;
            }

            visited[next] = true;
            dist[next] = dist[current] + 1;
            prev[next] = current;
            EnQueue(&queue, next);
        }
    }
}
```

如果采用邻接矩阵，BFS 扫描邻接点时需要看整行，时间复杂度为 $O(\lvert V\rvert^2)$；如果采用邻接表，总时间复杂度为 $O(\lvert V\rvert+\lvert E\rvert)$。

## Dijkstra：非负权图的单源最短路径

Dijkstra 用来求一个源点到其他各顶点的最短路径。它的核心思想是：每一轮从“尚未确定最短路径”的顶点里，选出当前 `dist` 最小的顶点，并把它加入已确定集合。

为什么这个顶点可以被确定？因为边权非负。若当前未确定顶点中 `dist[x]` 已经最小，那么任何绕路都必须先到达另一个未确定顶点，而到另一个未确定顶点的距离不会比 `dist[x]` 更小；再加上一条非负边，也不可能把到 `x` 的距离降下来。

这也是 Dijkstra 不能处理负权边的根本原因：负权边可能在后面把已经确定的距离变小。

[html-card height=860](../assets/shortest-path-dijkstra.html)

### 数组含义

- `final[v]`：源点到 `v` 的最短路径是否已经确定。
- `dist[v]`：当前已知的源点到 `v` 的最短距离。
- `prev[v]`：当前最短路径上 `v` 的前驱顶点。

初始化时：

- `final[source] = true`。
- `dist[source] = 0`。
- 对其他顶点 `v`，若源点到 `v` 有直接边，则 `dist[v]` 为该边权值，`prev[v] = source`；否则 `dist[v] = INF`，`prev[v] = -1`。

每一轮做两件事：

1. 在 `final[v] == false` 的顶点中找到 `dist[v]` 最小者 `u`。
2. 用 `u` 作为新的中转点，检查所有从 `u` 出发的边，尝试更新其他顶点的 `dist` 和 `prev`。

### C 代码

```c
#include <stdbool.h>
#include <stdio.h>

#define MAX_VERTEX_NUM 100
#define INF 0x3f3f3f3f

typedef struct {
    int vexNum;
    int edgeNum;
    int arcs[MAX_VERTEX_NUM][MAX_VERTEX_NUM];
} MGraph;

/**
 * Uses Dijkstra to compute shortest paths from one source.
 *
 * Args:
 *   graph: Weighted adjacency matrix. INF means no edge.
 *   source: The source vertex index.
 *   dist: Output array. dist[v] is the shortest distance from source to v.
 *   prev: Output array. prev[v] is the predecessor of v on the shortest path.
 *
 * Notes:
 *   This algorithm requires all edge weights to be non-negative.
 */
void Dijkstra(const MGraph *graph, int source, int dist[], int prev[]) {
    bool final[MAX_VERTEX_NUM] = {false};

    for (int v = 0; v < graph->vexNum; ++v) {
        dist[v] = graph->arcs[source][v];

        if (graph->arcs[source][v] < INF && v != source) {
            prev[v] = source;
        } else {
            prev[v] = -1;
        }
    }

    dist[source] = 0;
    prev[source] = -1;
    final[source] = true;

    for (int round = 1; round < graph->vexNum; ++round) {
        int minDist = INF;
        int selected = -1;

        /* Pick the unsettled vertex with the smallest current distance. */
        for (int v = 0; v < graph->vexNum; ++v) {
            if (!final[v] && dist[v] < minDist) {
                minDist = dist[v];
                selected = v;
            }
        }

        if (selected == -1) {
            break;
        }

        final[selected] = true;

        /* Relax all outgoing edges of the selected vertex. */
        for (int v = 0; v < graph->vexNum; ++v) {
            if (final[v] || graph->arcs[selected][v] == INF) {
                continue;
            }

            if (dist[selected] + graph->arcs[selected][v] < dist[v]) {
                dist[v] = dist[selected] + graph->arcs[selected][v];
                prev[v] = selected;
            }
        }
    }
}
```

邻接矩阵版本每一轮要扫描所有顶点找最小 `dist`，再扫描一行做松弛；共进行约 $\lvert V\rvert-1$ 轮，所以时间复杂度为 $O(\lvert V\rvert^2)$，适合稠密图。

### 负权边为什么会破坏 Dijkstra

设有边：

```text
V0 -> V1: 10
V0 -> V2: 7
V1 -> V2: -5
```

Dijkstra 会先确定 `V2`，因为当前 `dist[2] = 7 < dist[1] = 10`。但真实最短路径是：

```text
V0 -> V1 -> V2
```

长度为 $10 + (-5) = 5$。也就是说，`V2` 在被确定后又可能被负权边改小，这违背了 Dijkstra 的贪心前提。

## Floyd：每对顶点间的最短路径

先只允许顶点 `0` 作为中转站。对任意两个顶点 `i`、`j`，比较：

```text
不经过 0：dist[i][j]
经过 0：dist[i][0] + dist[0][j]
```

如果经过 0 更短，就更新 `dist[i][j]`。

然后把这个思路推广：

- 只允许 `V0` 中转。
- 允许 `V0, V1` 中转。
- 允许 `V0, V1, V2` 中转。
- 直到允许所有顶点中转。

这就是 Floyd 的动态规划思想。

[html-card height=1280](../assets/shortest-path-floyd.html)

### 递推含义

令 $A^{(k)}[i][j]$ 表示：从 $V_i$ 到 $V_j$ 的路径中，只允许经过 $V_0,V_1,\dots,V_k$ 作为中转点时的最短路径长度。

则有：

$$
A^{(k)}[i][j]=\min(A^{(k-1)}[i][j], A^{(k-1)}[i][k]+A^{(k-1)}[k][j])
$$

这里的判断很朴素：允许 `Vk` 之后，`i` 到 `j` 的最短路只有两种可能：

- 不经过 `Vk`，维持原值。
- 经过 `Vk`，拆成 `i -> k` 和 `k -> j` 两段。

### `path` 矩阵

`path[i][j]` 记录的是：当前已知的 `i -> j` 最短路径中，最后一次让路径变短的中转点。

例如 `path[0][2] = 1` 表示：从 `V0` 到 `V2` 的最短路径需要经过 `V1`。恢复完整路径时，可以递归地拆分：

```text
PrintPath(i, j):
    k = path[i][j]
    若 k == -1，说明 i 和 j 之间不需要继续拆分
    否则先恢复 i -> k，再恢复 k -> j
```

### C 代码

```c
#include <stdio.h>

#define MAX_VERTEX_NUM 100
#define INF 0x3f3f3f3f

/**
 * Uses Floyd-Warshall to compute all-pairs shortest paths.
 *
 * Args:
 *   vertexCount: Number of vertices.
 *   dist: Input and output matrix. dist[i][j] is the shortest distance i -> j.
 *   path: Output matrix. path[i][j] stores an intermediate vertex, or -1.
 *
 * Notes:
 *   dist[i][i] should be initialized to 0.
 *   dist[i][j] should be initialized to INF when there is no edge i -> j.
 */
void Floyd(int vertexCount,
           int dist[MAX_VERTEX_NUM][MAX_VERTEX_NUM],
           int path[MAX_VERTEX_NUM][MAX_VERTEX_NUM]) {
    for (int i = 0; i < vertexCount; ++i) {
        for (int j = 0; j < vertexCount; ++j) {
            path[i][j] = -1;
        }
    }
    
    // Floyd的核心就是这三层循环
    for (int k = 0; k < vertexCount; ++k) {
    // 之前轮次允许0~k-1作为中转站，这一轮新加入k
        for (int i = 0; i < vertexCount; ++i) {
            for (int j = 0; j < vertexCount; ++j) {
                if (dist[i][k] == INF || dist[k][j] == INF) {
                    continue;
                }
				// k作为新的中转站可以使得i-j距离进一步减少
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                    path[i][j] = k;
                }
            }
        }
    }
}

/**
 * Prints intermediate vertices on the shortest path from start to end.
 *
 * Args:
 *   path: Floyd path matrix.
 *   start: Start vertex.
 *   end: End vertex.
 */
void PrintFloydPath(int path[MAX_VERTEX_NUM][MAX_VERTEX_NUM], int start, int end) {
    int middle = path[start][end];

    if (middle == -1) {
        return;
    }

    PrintFloydPath(path, start, middle);
    printf("V%d ", middle);
    PrintFloydPath(path, middle, end);
}
```

Floyd 的时间复杂度为 $O(\lvert V\rvert^3)$，空间复杂度为 $O(\lvert V\rvert^2)$。**它可以处理负权边，但不能处理负权回路**。

若存在负权回路，路径可以绕着该回路反复走，使总长度不断变小，此时“最短路径”本身就可能没有定义。

也可以对每个顶点各运行一次 Dijkstra 来求每对顶点间的最短路径。若使用邻接矩阵朴素实现，总复杂度同样是 $O(\lvert V\rvert^3)$，但前提仍然是图中没有负权边。

## SPFA：允许负权边的单源最短路径

SPFA 可以理解为“只处理有希望继续变短的顶点”的 Bellman-Ford 队列优化。

普通松弛的想法是：如果已经知道 `dist[u]`，并且有边 `u -> v`，那么就检查：

```text
dist[u] + weight(u, v) < dist[v]
```

如果成立，说明通过 `u` 到 `v` 更短，于是更新 `dist[v]`。一旦 `dist[v]` 变短，`v` 的出边也可能让别的顶点继续变短，所以把 `v` 放进队列。

SPFA 的优势是写法直观，而且可以顺便**检测**负权回路；但它的最坏时间复杂度仍然可能达到 $O(\lvert V\rvert\lvert E\rvert)$。

[html-card height=980](../assets/shortest-path-spfa.html)

### 数组含义

- `dist[v]`：源点到 `v` 的当前最短距离。
- `prev[v]`：当前最短路径上 `v` 的前驱顶点。
- `inQueue[v]`：`v` 是否已经在队列中，避免重复入队。
- `enqueueCount[v]`：`v` 的入队次数。若某顶点入队次数超过 $\lvert V\rvert$，通常可判断存在负权回路。

### C 代码

```c
#include <stdbool.h>
#include <stdio.h>

#define MAX_VERTEX_NUM 100
#define MAX_EDGE_NUM 1000
#define INF 0x3f3f3f3f

typedef struct EdgeNode {
    int to;
    int weight;
    int next;
} EdgeNode;

typedef struct {
    int vexNum;
    int edgeNum;
    int head[MAX_VERTEX_NUM];
    EdgeNode edges[MAX_EDGE_NUM];
} ALGraph;

typedef struct {
    int data[MAX_VERTEX_NUM * MAX_VERTEX_NUM];
    int front;
    int rear;
} Queue;

static void InitQueue(Queue *queue) {
    queue->front = 0;
    queue->rear = 0;
}

static bool IsQueueEmpty(const Queue *queue) {
    return queue->front == queue->rear;
}

static void EnQueue(Queue *queue, int vertex) {
    queue->data[queue->rear++] = vertex;
}

static int DeQueue(Queue *queue) {
    return queue->data[queue->front++];
}

void InitALGraph(ALGraph *graph, int vertexCount) {
    graph->vexNum = vertexCount;
    graph->edgeNum = 0;

    for (int v = 0; v < vertexCount; ++v) {
        graph->head[v] = -1;
    }
}

void AddDirectedEdge(ALGraph *graph, int from, int to, int weight) {
    int edgeIndex = graph->edgeNum++;

    graph->edges[edgeIndex].to = to;
    graph->edges[edgeIndex].weight = weight;
    graph->edges[edgeIndex].next = graph->head[from];
    graph->head[from] = edgeIndex;
}

/**
 * Uses SPFA to compute shortest paths from one source.
 *
 * Args:
 *   graph: Directed weighted adjacency list.
 *   source: The source vertex index.
 *   dist: Output array. dist[v] is the shortest distance from source to v.
 *   prev: Output array. prev[v] is the predecessor of v on the shortest path.
 *
 * Returns:
 *   true if no negative cycle is found from source.
 *   false if a negative cycle is reachable from source.
 */
bool Spfa(const ALGraph *graph, int source, int dist[], int prev[]) {
    bool inQueue[MAX_VERTEX_NUM] = {false};
    int enqueueCount[MAX_VERTEX_NUM] = {0};
    Queue queue;
    InitQueue(&queue);

    for (int v = 0; v < graph->vexNum; ++v) {
        dist[v] = INF;
        prev[v] = -1;
    }

    dist[source] = 0;
    EnQueue(&queue, source);
    inQueue[source] = true;
    enqueueCount[source] = 1;

    while (!IsQueueEmpty(&queue)) {
        int current = DeQueue(&queue);
        inQueue[current] = false;

        for (int edgeIndex = graph->head[current];
             edgeIndex != -1;
             edgeIndex = graph->edges[edgeIndex].next) {
            int next = graph->edges[edgeIndex].to;
            int weight = graph->edges[edgeIndex].weight;

            if (dist[current] == INF || dist[current] + weight >= dist[next]) {
                continue;
            }

            dist[next] = dist[current] + weight;
            prev[next] = current;

            if (!inQueue[next]) {
                EnQueue(&queue, next);
                inQueue[next] = true;
                ++enqueueCount[next];

                if (enqueueCount[next] > graph->vexNum) {
                    return false;
                }
            }
        }
    }

    return true;
}
```

## 四种方法对比

| 方法       | 主要用途        | 边权限制              | 常用存储     | 时间复杂度                                                             |
| -------- | ----------- | ----------------- | -------- | ----------------------------------------------------------------- |
| BFS      | 无权图的单源最短路径  | 无权，或所有边权相同        | 邻接表或邻接矩阵 | 邻接表 $O(\lvert V\rvert+\lvert E\rvert)$，邻接矩阵 $O(\lvert V\rvert^2)$ |
| Dijkstra | 带权图的单源最短路径  | 不能有负权边            | 邻接矩阵常见   | $O(\lvert V\rvert^2)$                                             |
| Floyd    | 每对顶点间的最短路径  | 可有负权边，不能有负权回路     | 邻接矩阵     | $O(\lvert V\rvert^3)$                                             |
| SPFA     | 带负权边的单源最短路径 | 可有负权边，可**检测**负权回路 | 邻接表      | 最坏 $O(\lvert V\rvert\lvert E\rvert)$                              |

> [!tip] 选择题常考
> Dijkstra 不是“不能处理负权回路”这么简单，而是**只要存在负权边，就不满足它的正确性前提**。
> Floyd 可以处理负权边，但不能处理负权回路。

## 路径恢复

最短路径算法通常不只要求距离，还会要求具体路径。核心做法都是保存前驱或中转信息。

### 单源最短路径

BFS、Dijkstra、SPFA 常用 `prev[]`：

```text
prev[v] = u
```

表示当前最短路径上，`v` 的前一个顶点是 `u`。恢复路径时从终点反向追溯到源点，再逆序输出。

### Floyd

Floyd 常用 `path[][]`：

```text
path[i][j] = k
```

表示 `i -> j` 的最短路径需要经过中转点 `k`。恢复路径时递归拆成：

```text
i -> k
k -> j
```

直到某一段不再需要中转点。
