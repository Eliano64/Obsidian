---
title: AOE Network And Critical Path
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 23:08:01
katex: true
---

# AOE Network And Critical Path

AOE 网用于分析工程项目中“活动耗时”与“工程最短完成时间”。它和 [[DAG-and-AOV-network#AOV 网|AOV]] 都依赖有向无环结构，但含义不同：

- AOV 网：顶点表示活动，边表示活动先后约束，主要用于 [[topological-sorting|拓扑排序]]。
- AOE 网：顶点表示事件，边表示活动，边权表示活动耗时，主要用于求关键路径。

## AOE 网

在带权有向图中：

- 顶点表示**事件**。
- 有向边表示**活动**。
- 边上的权值表示完成该活动的开销，常见含义是活动所需时间。

这样的网络称为 AOE 网，即 Activity On Edge Network。

AOE 网有两个基本性质：

1. 只有某顶点所代表的事件发生后，从该顶点出发的活动才能开始。
2. 只有进入某顶点的各活动都已经结束时，该顶点所代表的事件才能发生。

AOE 网中通常只有一个入度为 0 的顶点，称为**源点**或开始顶点；只有一个出度为 0 的顶点，称为**汇点**或结束顶点。

## 关键路径

从源点到汇点的有向路径可能有多条。所有路径中，路径长度最大的路径称为**关键路径**，关键路径上的活动称为**关键活动**。

这里的路径长度是路径上活动耗时之和。

> [!important] 为什么是最长路径
> AOE 网允许活动并行。整个工程要等所有必要活动都完成才能结束，因此决定工程最短完成时间的是从源点到汇点耗时最长的那条依赖链，而不是最短路径。

关键路径的长度就是整个工程的最短完成时间。若关键活动延期，整个工程完成时间会延期。

## 四个时间量

### 事件最早发生时间 `ve(k)`

`ve(k)` 表示事件 `Vk` 最早可能发生的时间。它决定所有从 `Vk` 出发的活动最早什么时候能开始。

按[[topological-sorting#拓扑序列的定义|拓扑序列]]计算：

```text
ve(source) = 0
ve(k) = max{ve(j) + weight(j, k)}
```

其中 `Vj` 是 `Vk` 的任意前驱。

### 事件最迟发生时间 `vl(k)`

`vl(k)` 表示在不推迟整个工程完成的前提下，事件 `Vk` 最迟必须发生的时间。

按[[topological-sorting#逆拓扑排序|逆拓扑序列]]计算：

```text
vl(sink) = ve(sink)
vl(k) = min{vl(j) - weight(k, j)}
```

其中 `Vj` 是 `Vk` 的任意后继。

### 活动最早开始时间 `e(i)`

若活动 `ai` 对应边 `<Vk, Vj>`，则：

```text
e(i) = ve(k)
```

活动最早开始时间就是它的起点事件最早发生时间。

### 活动最迟开始时间 `l(i)`

若活动 `ai` 对应边 `<Vk, Vj>`，耗时为 `weight(k, j)`，则：

```text
l(i) = vl(j) - weight(k, j)
```

活动必须在终点事件最迟发生时间之前完成，所以最迟开始时间等于终点事件的 `vl` 减去活动耗时。

### 活动时间余量 `d(i)`

```text
d(i) = l(i) - e(i)
```

`d(i)` 表示活动 `ai` 在不增加整个工程工期的情况下，可以拖延的时间。

若：

```text
d(i) = 0
```

则该活动没有任何拖延余地，是关键活动。

## 求关键路径的完整步骤

1. 求拓扑序列。
2. 按拓扑序列求所有事件的 `ve[]`。
3. 按逆拓扑序列求所有事件的 `vl[]`。
4. 对每条活动边求 `e(i)`。
5. 对每条活动边求 `l(i)`。
6. 求 `d(i)=l(i)-e(i)`。
7. `d(i)=0` 的活动是关键活动，由关键活动组成关键路径。

时间复杂度与遍历图一样。

## 例题

> [!example]
> ![1000](../assets/aoe-critical-path-overview.svg)


### 求 `ve[]`

拓扑序列取：

```text
V1, V3, V2, V5, V4, V6
```

[html-card height=820](../assets/critical-path-ve.html)

计算过程：

```text
ve(1) = 0
ve(3) = ve(1) + 2 = 2
ve(2) = ve(1) + 3 = 3
ve(5) = ve(2) + 3 = 6
ve(4) = max{ve(2)+2, ve(3)+4} = max{5, 6} = 6
ve(6) = max{ve(5)+1, ve(4)+2, ve(3)+3} = max{7, 8, 5} = 8
```

`ve(6)` 表示工程最早完成时间。本例中关键候选来自 `V4 -> V6`，得到 `6+2=8`。

### 求 `vl[]`

逆拓扑序列取：

```text
V6, V5, V4, V2, V3, V1
```

[html-card height=820](../assets/critical-path-vl.html)

计算过程：

```text
vl(6) = ve(6) = 8
vl(5) = vl(6) - 1 = 7
vl(4) = vl(6) - 2 = 6
vl(2) = min{vl(5)-3, vl(4)-2} = min{4, 4} = 4
vl(3) = min{vl(4)-4, vl(6)-3} = min{2, 5} = 2
vl(1) = min{vl(2)-3, vl(3)-2} = min{1, 0} = 0
```

### 求活动的 `e`、`l`、`d`

[html-card height=820](../assets/critical-path-activities.html)

| 活动 | 边 | 耗时 | `e(i)` | `l(i)` | `d(i)` | 是否关键 |
|---|---|---:|---:|---:|---:|---|
| `a1` | `V1 -> V2` | 3 | 0 | 1 | 1 | 否 |
| `a2` | `V1 -> V3` | 2 | 0 | 0 | 0 | 是 |
| `a3` | `V2 -> V4` | 2 | 3 | 4 | 1 | 否 |
| `a4` | `V2 -> V5` | 3 | 3 | 4 | 1 | 否 |
| `a5` | `V3 -> V4` | 4 | 2 | 2 | 0 | 是 |
| `a6` | `V3 -> V6` | 3 | 2 | 5 | 3 | 否 |
| `a7` | `V4 -> V6` | 2 | 6 | 6 | 0 | 是 |
| `a8` | `V5 -> V6` | 1 | 6 | 7 | 1 | 否 |

因此关键活动是：

```text
a2, a5, a7
```

关键路径是：

```text
V1 -> V3 -> V4 -> V6
```

关键路径长度为：

```text
2 + 4 + 2 = 8
```

## C 代码

下面代码使用邻接表保存 AOE 网。拓扑排序用于计算 `ve[]`；逆向扫描拓扑序列用于计算 `vl[]`；最后遍历所有边求活动的 `e`、`l`、`d`。

```c
#include <stdbool.h>
#include <stdio.h>

#define MAX_VERTEX_NUM 100
#define MAX_EDGE_NUM 1000
#define INF 0x3f3f3f3f

typedef struct EdgeNode {
    int from;
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
    int data[MAX_VERTEX_NUM];
    int front;
    int rear;
} Queue;

void InitGraph(ALGraph *graph, int vertexCount) {
    graph->vexNum = vertexCount;
    graph->edgeNum = 0;

    for (int v = 0; v < vertexCount; ++v) {
        graph->head[v] = -1;
    }
}

void AddActivity(ALGraph *graph, int from, int to, int weight) {
    int edgeIndex = graph->edgeNum++;

    graph->edges[edgeIndex].from = from;
    graph->edges[edgeIndex].to = to;
    graph->edges[edgeIndex].weight = weight;
    graph->edges[edgeIndex].next = graph->head[from];
    graph->head[from] = edgeIndex;
}

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
 * Computes topological order and event earliest times.
 *
 * Args:
 *   graph: AOE network stored by adjacency list.
 *   topoOrder: Output topological sequence.
 *   ve: Output earliest occurrence time of each event.
 *
 * Returns:
 *   true if the network is acyclic.
 *   false if a directed cycle exists.
 */
bool TopologicalOrderAndVe(const ALGraph *graph, int topoOrder[], int ve[]) {
    int inDegree[MAX_VERTEX_NUM] = {0};
    Queue queue;
    int count = 0;

    InitQueue(&queue);

    for (int v = 0; v < graph->vexNum; ++v) {
        ve[v] = 0;
    }

    for (int i = 0; i < graph->edgeNum; ++i) {
        ++inDegree[graph->edges[i].to];
    }

    for (int v = 0; v < graph->vexNum; ++v) {
        if (inDegree[v] == 0) {
            EnQueue(&queue, v);
        }
    }

    while (!IsEmpty(&queue)) {
        int current = DeQueue(&queue);
        topoOrder[count++] = current;

        for (int edgeIndex = graph->head[current];
             edgeIndex != -1;
             edgeIndex = graph->edges[edgeIndex].next) {
            int next = graph->edges[edgeIndex].to;
            int weight = graph->edges[edgeIndex].weight;

            if (ve[current] + weight > ve[next]) {
                ve[next] = ve[current] + weight;
            }

            --inDegree[next];
            if (inDegree[next] == 0) {
                EnQueue(&queue, next);
            }
        }
    }

    return count == graph->vexNum;
}

/**
 * Computes latest occurrence times and prints critical activities.
 *
 * Args:
 *   graph: AOE network.
 *   topoOrder: Topological sequence.
 *   ve: Earliest occurrence time of each event.
 *   vl: Output latest occurrence time of each event.
 */
void CriticalPath(const ALGraph *graph,
                  const int topoOrder[],
                  const int ve[],
                  int vl[]) {
    int sink = topoOrder[graph->vexNum - 1];

    for (int v = 0; v < graph->vexNum; ++v) {
        vl[v] = ve[sink];
    }

    for (int i = graph->vexNum - 1; i >= 0; --i) {
        int current = topoOrder[i];

        for (int edgeIndex = graph->head[current];
             edgeIndex != -1;
             edgeIndex = graph->edges[edgeIndex].next) {
            int next = graph->edges[edgeIndex].to;
            int weight = graph->edges[edgeIndex].weight;

            if (vl[next] - weight < vl[current]) {
                vl[current] = vl[next] - weight;
            }
        }
    }

    for (int i = 0; i < graph->edgeNum; ++i) {
        int from = graph->edges[i].from;
        int to = graph->edges[i].to;
        int weight = graph->edges[i].weight;
        int earliest = ve[from];
        int latest = vl[to] - weight;
        int slack = latest - earliest;

        if (slack == 0) {
            printf("critical activity: V%d -> V%d, weight=%d\n",
                   from, to, weight);
        }
    }
}
```

## 性质与易错点

- 关键路径可能不唯一。
- 关键活动耗时增加，整个工程工期一定增加。
- 缩短关键活动时间可能缩短工期，但缩短到一定程度后，原关键活动可能变为非关键活动。
- 若存在多条关键路径，只缩短其中一条关键路径上的关键活动，不能缩短整个工期；必须缩短公共的关键活动。
- 求 `ve[]` 用拓扑序列；求 `vl[]` 用逆拓扑序列。顺序不能反。
- `e(i)` 和 `l(i)` 是活动的时间；`ve(k)` 和 `vl(k)` 是事件的时间。不要混用。
