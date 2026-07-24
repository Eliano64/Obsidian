---
title: Singly Linked List Insert Delete
tags:
  - LinearList
categories:
  - Data Structure & Algorithm
date: 2026-06-27 13:13:25
katex: true
---

# 单链表的插入与删除

## 按位序插入：带头结点

`ListInsert(&L, i, e)` 在第 `i` 个位置插入元素 `e`。

核心思路：找到第 `i - 1` 个结点 `prev`，将新结点 `newNode` 插入到 `prev` 之后。

[html-card](../assets/singly-linked-list-position-insert.html)

基本步骤：

1. 检查 `i` 是否合法。
2. 从头结点开始寻找第 `i - 1` 个结点。
3. 申请新结点 `newNode`，令 `newNode->data = e`。
4. 令 `newNode->next = prev->next`，先接住原后继。
5. 令 `prev->next = newNode`，再让前驱指向新结点。

时间复杂度：查找前驱结点需要 `O(n)`，真正修改指针为 `O(1)`。

```c
bool ListInsert(LinkList head, int index, ElemType value) {
    if (index < 1) return false;

    LNode *prev = head;
    int position = 0;
    while (prev != NULL && position < index - 1) {
        prev = prev->next;
        position++;
    }
    if (prev == NULL) return false;

    LNode *newNode = (LNode *)malloc(sizeof(LNode));
    if (newNode == NULL) return false;
    newNode->data = value;
    newNode->next = prev->next;  // 先接住原后继，避免断链
    prev->next = newNode;        // 再让前驱指向新结点
    return true;
}
```

带头结点时，`position = 0` 的 `prev` 是头结点。插入第 1 位就是找到第 0 个结点，也就是头结点，然后执行一次后插。

## 按位序插入：不带头结点

不带头结点时，若 `i = 1`，需要直接修改头指针 `L`，这是特殊情况；若 `i > 1`，仍需找到第 `i - 1` 个结点再后插。带头结点的价值就在于消除这种表头特殊处理。

## 指定结点的后插

已知结点 `prev`，在其后插入 `newNode`：

[html-card](../assets/singly-linked-list-insert-after.html)

```c
newNode->next = prev->next;
prev->next = newNode;
```

时间复杂度：`O(1)`。

两句不能交换。若先执行 `prev->next = newNode`，原来 `prev` 的后继地址会被覆盖，新结点就无法接回原链表后半段。

## 指定结点前插

单链表不能直接逆向找到前驱。若只给定结点 `node`，常用技巧是“后插 + 交换数据”：

[html-card](../assets/singly-linked-list-insert-before-copy.html)

1. 在 `node` 后插入新结点 `newNode`。
2. 将 `node` 的数据复制到 `newNode`。
3. 将待插入数据写入 `node`。

这样逻辑效果等价于在 `node` 前插入，时间复杂度为 `O(1)`。

## 按位序删除：带头结点

`ListDelete(&L, i, &e)` 删除第 `i` 个位置元素。

核心思路：找到第 `i - 1` 个结点 `prev`，令 `target = prev->next`，删除 `target`。

[html-card](../assets/linked-list-delete.html)

基本步骤：

1. 检查 `i` 是否合法。
2. 找到第 `i - 1` 个结点 `prev`。
3. 令 `target = prev->next`，保存被删结点指针。
4. 用 `deletedValue` 保存 `target->data`。
5. 令 `prev->next = target->next`，让前驱跳过目标结点。
6. 释放 `target`。

时间复杂度：查找前驱结点 `O(n)`，修改指针和释放结点 `O(1)`。

```c
bool ListDelete(LinkList head, int index, ElemType *deletedValue) {
    if (index < 1) return false;

    LNode *prev = head;
    int position = 0;
    while (prev != NULL && position < index - 1) {
        prev = prev->next;
        position++;
    }
    if (prev == NULL || prev->next == NULL) return false;

    LNode *target = prev->next;
    *deletedValue = target->data;
    prev->next = target->next;  // 先改链，保留 target 以便释放
    free(target);
    return true;
}
```

删除必须检查 `prev->next != NULL`。如果第 `i` 个结点不存在，直接访问 `prev->next->data` 会出错。

## 指定结点的删除

若已知结点 `node` 且 `node` 不是最后一个结点，可用后继结点覆盖法：

1. 令 `nextNode = node->next`。
2. 将 `nextNode->data` 复制到 `node->data`。
3. 令 `node->next = nextNode->next`。
4. 释放 `nextNode`。

局限：若 `node` 是最后一个结点，没有后继可覆盖，仍必须从头遍历找到 `node` 的前驱。

## 翻转给定区间

对于带头结点的单链表，翻转第 `left` 到第 `right` 个数据结点时，先找到第 `left - 1` 个结点 `before`。令 `first = before->next`，每轮取下 `first` 后面的结点，再把它插到 `before` 后面。`first` 不随结点移动，最后自然成为翻转区间的尾结点。

[html-card height=620 step=40](../assets/singly-linked-list-range-reverse.html)

实现如下。这里默认参数值合法。

```c
void ReverseBetween(LinkList head, int left, int right) {
    // 找到第 left - 1 个结点。
    LNode *before = head;
    for (int position = 1; position < left; ++position) {
        before = before->next;
    }

    LNode *first = before->next;
    // 每轮把 first 后面的结点摘下，插到 before 后面。
    for (int count = 0; count < right - left; ++count) {
        LNode *moved = first->next;
        first->next = moved->next;
        moved->next = before->next;
        before->next = moved;
    }
}
```

寻找区间并完成反转共需 `O(n)` 时间；算法只使用固定数量的指针，额外空间为 `O(1)`。当 `left = right` 时循环执行零次，链表保持不变。

## 关联

顺序表插入删除的主要成本是移动元素，见 [[sequential-list-insert-delete|顺序表的插入与删除]]；单链表的主要成本是查找前驱。
