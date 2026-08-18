---
title: Topological Sorting
tags:
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 22:04:10
katex: true
---

# 拓扑序列的定义

一个有向图的顶点序列若满足：

1. 每个顶点出现且只出现一次。
2. 若图中存在从顶点 `A` 到顶点 `B` 的路径，则 `A` 必须排在 `B` 前面。

则称该序列为这个图的一个**拓扑序列**。

如果序列中 `A` 在 `B` 前面，那么图中不能存在从 `B` 到 `A` 的路径。显然，**拓扑序列存在的充要条件是该图是有向无环图。**

> [!note] 和 DFS 判环的关系
> 拓扑排序本身可以判断有向图是否有环。DFS 也可以通过 `VISITING` 状态判断有向环，见[[Depth-First-Search#DFS 判断有向图是否有环|DFS 判有向图环]]。

## 拓扑序列唯一的充要条件

若一个有向无环图存在一条能够访问所有顶点的路径，则该图的拓扑序列唯一。且该路径即为拓扑序列。

# 得到拓扑序列的算法

## Kahn 算法：不断删除入度为 0 的顶点

Kahn 算法是考研中最常见的拓扑排序实现。时间复杂度与遍历图一样。

核心规则：

1. 从 AOV 网中选择一个没有前驱的顶点，也就是入度为 0 的顶点，并输出。
2. 从图中删除该顶点以及所有以它为起点的有向边。
3. 重复以上过程，直到图为空；若图未空但已经找不到入度为 0 的顶点，则说明图中存在有向环。

[html-card height=760](../assets/topological-sorting-kahn.html)

### 为什么入度为 0 的顶点可以输出

入度为 0 表示当前图中没有任何活动必须排在它前面。因此它可以作为当前最先执行的活动。

输出一个入度为 0 的顶点后，要删除它的所有出边。这一步的含义是：该活动已经完成，它对后继活动的前置约束已经满足。所以后继顶点的入度要减 1。

### 有环时为什么会停住

如果当前图中还有顶点，但没有入度为 0 的顶点，说明剩下的每个顶点都有前驱。沿着前驱不断往前找，由于顶点数有限，最终一定会重复遇到某个顶点，从而形成有向环。

[html-card height=700](../assets/topological-sorting-cycle.html)

## C 代码

删除某个顶点的出边时，需要扫描它的出边，并把后继顶点的入度减 1。

以下以邻接链表存储图。

```c
#include <stdbool.h>
#include <stdio.h>

#define MAX_VERTEX_NUM 100
#define MAX_EDGE_NUM 1000

typedef struct EdgeNode {
    int to;
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
    int top;
} Stack;

void InitGraph(ALGraph *graph, int vertexCount) {
    graph->vexNum = vertexCount;
    graph->edgeNum = 0;

    for (int v = 0; v < vertexCount; ++v) {
        graph->head[v] = -1;
    }
}

void AddDirectedEdge(ALGraph *graph, int from, int to) {
    int edgeIndex = graph->edgeNum++;

    graph->edges[edgeIndex].to = to;
    graph->edges[edgeIndex].next = graph->head[from];
    graph->head[from] = edgeIndex;
}

static void InitStack(Stack *stack) {
    stack->top = -1;
}

static bool IsEmpty(const Stack *stack) {
    return stack->top == -1;
}

static void Push(Stack *stack, int vertex) {
    stack->data[++stack->top] = vertex;
}

static int Pop(Stack *stack) {
    return stack->data[stack->top--];
}

/**
 * Computes a topological order by Kahn's algorithm.
 *
 * Args:
 *   graph: Directed graph stored by adjacency list.
 *   topoOrder: Output array. topoOrder[i] is the i-th vertex in the order.
 *
 * Returns:
 *   true if a topological order exists.
 *   false if the graph contains a directed cycle.
 */
bool TopologicalSort(const ALGraph *graph, int topoOrder[]) {
    int inDegree[MAX_VERTEX_NUM] = {0};
    Stack zeroInDegree;
    int count = 0;

    InitStack(&zeroInDegree);

    /* Compute in-degree of every vertex by scanning all outgoing edges. */
    for (int from = 0; from < graph->vexNum; ++from) {
        for (int edgeIndex = graph->head[from];
             edgeIndex != -1;
             edgeIndex = graph->edges[edgeIndex].next) {
            int to = graph->edges[edgeIndex].to;
            ++inDegree[to];
        }
    }

    /* Vertices with in-degree 0 can be output immediately. */
    for (int v = 0; v < graph->vexNum; ++v) {
        if (inDegree[v] == 0) {
            Push(&zeroInDegree, v);
        }
    }

    while (!IsEmpty(&zeroInDegree)) {
        int current = Pop(&zeroInDegree);
        topoOrder[count++] = current;

        /* Deleting current means all its outgoing constraints are removed. */
        for (int edgeIndex = graph->head[current];
             edgeIndex != -1;
             edgeIndex = graph->edges[edgeIndex].next) {
            int next = graph->edges[edgeIndex].to;
            --inDegree[next];

            if (inDegree[next] == 0) {
                Push(&zeroInDegree, next);
            }
        }
    }

    return count == graph->vexNum;
}
```

如果用栈保存入度为 0 的顶点，输出顺序会受入栈顺序影响；如果用队列，也会得到另一个合法序列。

由于需要扫描每一个点的每一条出边，所以实现的时间复杂度相当于遍历了图，与图的存储方式有关；空间复杂度只与点有关，为 $O(\lvert V\rvert)$ 。

# 逆拓扑排序

逆拓扑排序是从后往前安排活动。

操作规则：

1. 选择一个没有后继的顶点，也就是出度为 0 的顶点，并输出。
2. 删除该顶点以及所有以它为终点的有向边。
3. 重复直到图为空。

时间复杂度与遍历图一样。

它得到的是拓扑序列的反方向。若一个序列是合法拓扑序列，那么把它倒过来就是合法逆拓扑序列。

## DFS 实现逆拓扑排序

DFS 也可以得到逆拓扑序列：对一个顶点做 DFS 时，先递归访问它的所有后继；当这个顶点的所有后继都处理完，准备从递归栈退出时，再输出该顶点。

[html-card height=740](../assets/reverse-topological-dfs.html)

## C 代码

```c
#include <stdbool.h>
#include <stdio.h>

#define MAX_VERTEX_NUM 100
#define MAX_EDGE_NUM 1000

typedef enum {
    UNVISITED,
    VISITING,
    DONE
} VisitState;

typedef struct EdgeNode {
    int to;
    int next;
} EdgeNode;

typedef struct {
    int vexNum;
    int edgeNum;
    int head[MAX_VERTEX_NUM];
    EdgeNode edges[MAX_EDGE_NUM];
} ALGraph;

/**
 * DFS helper for reverse topological order.
 *
 * Args:
 *   graph: Directed graph stored by adjacency list.
 *   vertex: Current vertex.
 *   state: Visit states for cycle detection.
 *   reverseOrder: Output array for reverse topological order.
 *   count: Number of vertices already written into reverseOrder.
 *
 * Returns:
 *   true if no directed cycle is found in this DFS branch.
 *   false if an edge points to a VISITING vertex, which means a cycle exists.
 */
static bool DfsReverseTopo(const ALGraph *graph,
                           int vertex,
                           VisitState state[],
                           int reverseOrder[],
                           int *count) {
    state[vertex] = VISITING;

    for (int edgeIndex = graph->head[vertex];
         edgeIndex != -1;
         edgeIndex = graph->edges[edgeIndex].next) {
        int next = graph->edges[edgeIndex].to;

        if (state[next] == VISITING) {
            return false;
        }

        if (state[next] == UNVISITED) {
            if (!DfsReverseTopo(graph, next, state, reverseOrder, count)) {
                return false;
            }
        }
    }

    state[vertex] = DONE;
    reverseOrder[(*count)++] = vertex;
    return true;
}

/**
 * Computes reverse topological order by DFS.
 *
 * Args:
 *   graph: Directed graph stored by adjacency list.
 *   reverseOrder: Output array. Vertices are written in reverse topological order.
 *
 * Returns:
 *   true if the graph is a DAG.
 *   false if a directed cycle exists.
 */
bool ReverseTopologicalSortByDfs(const ALGraph *graph, int reverseOrder[]) {
    VisitState state[MAX_VERTEX_NUM];
    int count = 0;

    for (int v = 0; v < graph->vexNum; ++v) {
        state[v] = UNVISITED;
    }

    for (int v = 0; v < graph->vexNum; ++v) {
        if (state[v] == UNVISITED) {
            if (!DfsReverseTopo(graph, v, state, reverseOrder, &count)) {
                return false;
            }
        }
    }

    return true;
}
```

DFS 版本要特别注意：如果遇到 `VISITING` 顶点，说明边指向当前递归栈中的祖先或正在处理的顶点，因此存在有向环；如果遇到 `DONE` 顶点，则只是指向一个已经完成的分支，不能误判为环。