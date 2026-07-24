---
title: Queue Applications
tags:
  - Queue
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:12:00
katex: true
---

# 队列的应用

[[queue-definition-and-operations|队列]] 的核心特征是先进先出，适合保存“先到先处理”的对象，也适合按层次、按距离逐步扩展的过程。

## 树的层次遍历

层次遍历从根结点开始，按从上到下、从左到右的顺序访问结点。队列保存“已经发现但还没访问其孩子”的结点。

```c
void LevelOrder(BiTree root) {
    if (root == NULL) return;

    LinkQueue queue;
    InitQueue(&queue);
    EnQueue(&queue, root);

    while (!QueueEmpty(queue)) {
        BiTNode *node;
        DeQueue(&queue, &node);
        Visit(node);

        if (node->lchild != NULL) EnQueue(&queue, node->lchild);
        if (node->rchild != NULL) EnQueue(&queue, node->rchild);
    }
}
```

队列保证先入队的上一层结点先被处理，因此孩子结点自然按层次展开。

## 图的广度优先遍历

广度优先遍历从某个顶点出发，先访问距离为 `1` 的邻接点，再访问距离为 `2` 的顶点，以此类推。队列保存“已访问但邻接点尚未扩展完”的顶点。

```c
void BFS(Graph graph, int start) {
    bool visited[MAX_VERTEX_NUM] = {false};
    LinkQueue queue;
    InitQueue(&queue);

    Visit(start);
    visited[start] = true;
    EnQueue(&queue, start);

    while (!QueueEmpty(queue)) {
        int vertex;
        DeQueue(&queue, &vertex);

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

若不使用 `visited` 标记，图中有环时会重复入队，甚至无法结束。

## 操作系统中的队列

操作系统中常用队列组织等待处理的对象：

- 就绪进程队列：等待 CPU 调度的进程按规则排队。
- 阻塞队列：等待某类事件或资源的进程排队。
- 缓冲区队列：例如打印任务或输入输出数据按到达顺序等待处理。

这些场景的共同点是：对象到达有先后，处理资源有限，需要保存等待顺序。

## 易错点

- 层次遍历和广度优先遍历都依赖队列，但一个面向树，一个面向图。
- 图的 BFS 必须配合访问标记；树无环，通常不需要 `visited`。
- 队列只保证先进先出，具体调度策略可能还会叠加优先级、时间片等规则。
