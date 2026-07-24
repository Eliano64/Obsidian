---
title: Linked Queue
tags:
  - Queue
categories:
  - Data Structure & Algorithm
date: 2026-06-27 15:24:00
katex: true
---

# 链队列

## 定义

链队列是用链式存储实现的 [[queue-definition-and-operations|队列]]。通常设置队头指针 `front` 和队尾指针 `rear`。

带头结点的链队列结构：

```c
typedef struct LinkNode {
    ElemType data;
    struct LinkNode *next;
} LinkNode;

typedef struct {
    LinkNode *front;
    LinkNode *rear;
} LinkQueue;
```

## 带头结点初始化

```c
bool InitQueue(LinkQueue *queue) {
    LinkNode *head = (LinkNode *)malloc(sizeof(LinkNode));
    if (head == NULL) return false;
    head->next = NULL;
    queue->front = head;
    queue->rear = head;
    return true;
}
```

带头结点时，空队列满足：

```c
queue.front == queue.rear
```

## 入队

入队发生在队尾：

```c
bool EnQueue(LinkQueue *queue, ElemType value) {
    LinkNode *newNode = (LinkNode *)malloc(sizeof(LinkNode));
    if (newNode == NULL) return false;

    newNode->data = value;
    newNode->next = NULL;
    queue->rear->next = newNode;
    queue->rear = newNode;
    return true;
}
```

先让原队尾接上新结点，再更新 `rear`。

## 出队

出队删除队头结点的后继：

```c
bool DeQueue(LinkQueue *queue, ElemType *value) {
    if (queue->front == queue->rear) return false;

    LinkNode *target = queue->front->next;
    *value = target->data;
    queue->front->next = target->next;

    if (queue->rear == target) {
        queue->rear = queue->front;
    }

    free(target);
    return true;
}
```

若删除的是最后一个数据结点，必须让 `rear` 回到头结点，否则 `rear` 会悬空。

## 不带头结点的特殊情况

不带头结点时，空队列通常满足：

```c
queue.front == NULL && queue.rear == NULL
```

第一次入队和删除最后一个结点都需要特殊处理。考试中若未说明，带头结点链队列更容易写出统一代码。
