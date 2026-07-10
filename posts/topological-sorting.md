---
title: Topological Sorting 
tags:
  - DataStructureAndAlgorithm
  - Graph
categories:
  - Data Structure & Algorithm
date: 2026-06-28 22:04:10 
katex: true
---

# Topological Sorting

拓扑排序解决的是 AOV 网中的“先后顺序”问题。

AOV 网用顶点表示活动，用有向边 `<u, v>` 表示活动 `u` 必须先于活动 `v` 进行。因为活动依赖不能循环，所以 AOV 网必须是 DAG，也就是有向无环图。

> [!note] 和 DFS 判环的关系
> 拓扑排序本身可以判断有向图是否有环。DFS 也可以通过 `VISITING` 状态判断有向环，见 [DFS 判有向图环](depth-first-search.md)。

## 拓扑序列的定义

一个有向图的顶点序列若满足：

1. 每个顶点出现且只出现一次。
2. 若图中存在从顶点 `A` 到顶点 `B` 的路径，则 `A` 必须排在 `B` 前面。

则称该序列为这个图的一个**拓扑序列**。

等价地说：如果序列中 `A` 在 `B` 前面，那么图中不能存在从 `B` 到 `A` 的路径。

拓扑序列通常不唯一。例如两个活动互不依赖时，它们谁先谁后都可以。

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

## C 代码：邻接表 + 入度数组

邻接表适合实现拓扑排序，因为删除某个顶点的出边时，只需要扫描它的出边链表，并把后继顶点的入度减 1。

这里用数组模拟邻接表，便于考试时写出完整结构。

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

如果用栈保存入度为 0 的顶点，输出顺序会受入栈顺序影响；如果用队列，也会得到另一个合法序列。因此，拓扑序列不唯一是正常现象。

邻接表实现的时间复杂度为 $O(\lvert V\rvert+\lvert E\rvert)$，空间复杂度为 $O(\lvert V\rvert)$ 加上图本身的存储空间。

## 逆拓扑排序

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

### C 代码

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

## 能否用遍历序列唯一确定 DAG

不能像二叉树那样，简单地用“多个遍历序列”唯一确定一个 DAG。

二叉树的先序、中序、后序序列之所以能组合出结构，是因为二叉树有很强的局部结构约束：每个结点最多有左、右两个孩子，中序序列还能把左右子树切开。

DAG 没有这种左右子树边界。一个顶点可以有多个前驱、多个后继，不同边集也可能产生同一个遍历序列或同一个拓扑序列。

### 单个拓扑序列只能确定顺序约束

若给出拓扑序列：

```text
A, B, C
```

则只能说明所有边都必须从左往右连，例如可以有：

```text
A -> B, B -> C
```

也可以有：

```text
A -> C
```

还可以有：

```text
A -> B, A -> C, B -> C
```

这些都是合法 DAG，且都允许 `A, B, C` 作为拓扑序列。因此，一个拓扑序列不能确定 DAG 的边集。

### BFS 和 DFS 序列也不能唯一确定 DAG

只给 BFS 序列或 DFS 序列，一般也不能唯一确定 DAG。原因是：

- BFS/DFS 序列依赖起点和邻接点扫描顺序。
- DAG 中不同边集可能让顶点以同样顺序被发现。
- 遍历序列记录的是“访问顺序”，不是“所有依赖边”。

例如序列 `A, B, C` 可能来自只有 `A -> B, A -> C` 的图，也可能来自还有 `B -> C` 的图。只看访问顺序无法判断 `B -> C` 是否存在。

### 如果给出所有拓扑序列呢

若知道一个 DAG 的**所有**拓扑序列，可以恢复顶点之间的**可达偏序**：

- 若 `u` 在所有拓扑序列中都排在 `v` 前面，则 DAG 中一定存在从 `u` 到 `v` 的路径。
- 若 `u` 和 `v` 没有可达关系，则一定存在某个拓扑序列让 `u` 在前，也存在某个拓扑序列让 `v` 在前。

但这仍然不能唯一确定边集。因为可达关系只能说明“存在路径”，不能说明这条路径是直接边还是经过中间顶点。

例如：

```text
A -> B -> C
```

和：

```text
A -> B -> C
A -> C
```

它们的所有拓扑序列都是：

```text
A, B, C
```

但第二个图比第一个图多了一条传递边 `A -> C`。所以即使知道所有拓扑序列，也通常只能确定可达关系或传递约简意义上的骨架，不能唯一确定原始 DAG。

> [!tip] 做题判断
> 若题目问“某序列是否可能是该 DAG 的拓扑序列”，就检查每条边是否都从序列靠前顶点指向靠后顶点。若题目问“由序列确定 DAG”，通常答案是否定的，除非题目额外给出完整邻接关系、扫描规则或其他足以确定边集的条件。
