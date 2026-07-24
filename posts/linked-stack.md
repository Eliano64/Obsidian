---
title: Linked Stack
tags:
  - Stack
categories:
  - Data Structure & Algorithm
date: 2026-06-27 15:24:00
katex: true
---

# 链栈

## 定义

链栈是用链式存储实现的 [[stack-definition-and-operations|栈]]。通常把链表头部作为栈顶，因为头插和头删都只需要 `O(1)`。

```c
typedef struct StackNode {
    ElemType data;
    struct StackNode *next;
} StackNode, *LinkStack;
```

不带头结点时，空栈可表示为：

```c
LinkStack top = NULL;
```

## 进栈

进栈等价于单链表头插：

```c
bool Push(LinkStack *top, ElemType value) {
    StackNode *newNode = (StackNode *)malloc(sizeof(StackNode));
    if (newNode == NULL) return false;

    newNode->data = value;
    newNode->next = *top;
    *top = newNode;
    return true;
}
```

先让 `newNode->next` 指向原栈顶，再更新 `top`，这样原栈不会丢失。

## 出栈

出栈等价于单链表头删：

```c
bool Pop(LinkStack *top, ElemType *value) {
    if (*top == NULL) return false;

    StackNode *target = *top;
    *value = target->data;
    *top = target->next;
    free(target);
    return true;
}
```

需要先保存原栈顶结点 `target`，否则更新 `top` 后不方便释放原结点。

## 特点

- 不需要预先确定最大容量。
- 结点需要额外指针域，空间开销比顺序栈大。
- 只要内存可申请，通常不考虑“栈满”；但申请失败仍要处理。

链栈和 [[singly-linked-list-insert-delete|单链表头插头删]] 的代码结构非常接近，区别在于链栈只暴露栈顶操作。
