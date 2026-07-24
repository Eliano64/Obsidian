---
title: Singly Linked List Search And Build
tags:
  - LinearList
categories:
  - Data Structure & Algorithm
date: 2026-06-27 13:13:25
katex: true
---

# 单链表的查找与建立

## 按位查找

带头结点单链表中，`GetElem(L, i)` 获取第 `i` 个数据结点。

基本思路：从头结点开始，沿 `next` 指针向后移动，直到第 `i` 个数据结点。

[html-card](../assets/singly-linked-list-index-search.html)

边界：

- `i < 1` 非法。
- 遍历到 `NULL` 仍未到第 `i` 个位置，说明位序越界。

时间复杂度：`O(n)`。

```c
LNode *GetElem(LinkList head, int index) {
    if (index < 1) return NULL;
    LNode *current = head->next;
    int position = 1;
    while (current != NULL && position < index) {
        current = current->next;
        position++;
    }
    return current;
}
```

返回 `NULL` 表示位序非法或越界。注意这里 `position = 1` 时，`current` 指向第一个数据结点，而不是头结点。

## 按值查找

从第一个数据结点开始，依次比较 `data` 与目标值 `e`。找到则返回该结点指针或位序，找不到返回 `NULL` 或失败标记。

[html-card](../assets/singly-linked-list-value-search.html)

时间复杂度：`O(n)`。

```c
LNode *LocateElem(LinkList head, ElemType value) {
    LNode *current = head->next;
    while (current != NULL && current->data != value) {
        current = current->next;
    }
    return current;
}
```

## 求表长

从第一个数据结点开始遍历，计数直到 `NULL`。

[html-card](../assets/singly-linked-list-length.html)

时间复杂度：`O(n)`。

```c
int Length(LinkList head) {
    int length = 0;
    LNode *current = head->next;
    while (current != NULL) {
        length++;
        current = current->next;
    }
    return length;
}
```

## 尾插法建立单链表

尾插法按输入顺序建立链表，适合保持原序。

[html-card](../assets/singly-linked-list-tail-build.html)

基本步骤：

1. 初始化带头结点链表。
2. 设置尾指针 `tail` 指向当前表尾，初始为头结点。
3. 每读入一个元素，申请新结点 `newNode`。
4. 令 `tail->next = newNode`，再令 `tail = newNode`。
5. 输入结束后令 `tail->next = NULL`。

特点：建立结果与输入顺序一致。

```c
LinkList List_TailInsert(void) {
    LinkList head = (LNode *)malloc(sizeof(LNode));
    head->next = NULL;
    LNode *tail = head;  // tail 始终指向当前表尾

    ElemType value;
    while (scanf("%d", &value) == 1) {
        LNode *newNode = (LNode *)malloc(sizeof(LNode));
        newNode->data = value;
        tail->next = newNode;
        tail = newNode;
    }
    tail->next = NULL;
    return head;
}
```

尾指针 `tail` 始终指向当前最后一个结点，避免每插入一个新结点都从头找表尾。

## 头插法建立单链表

头插法每次把新结点插入头结点之后。

[html-card](../assets/singly-linked-list-head-build.html)

基本步骤：

1. 初始化带头结点链表。
2. 每读入一个元素，申请新结点 `newNode`。
3. 令 `newNode->next = head->next`。
4. 令 `head->next = newNode`。

特点：建立结果与输入顺序相反；常用于链表逆置类问题。

```c
LinkList List_HeadInsert(void) {
    LinkList head = (LNode *)malloc(sizeof(LNode));
    head->next = NULL;

    ElemType value;
    while (scanf("%d", &value) == 1) {
        LNode *newNode = (LNode *)malloc(sizeof(LNode));
        newNode->data = value;
        newNode->next = head->next;  // 先接住原首元结点
        head->next = newNode;        // 新结点成为新的首元结点
    }
    return head;
}
```

头插法每次都插在头结点之后，所以后读入的元素会排在更前面。

## 关联

建立链表本质上反复使用后插操作。插入和删除细节见 [[singly-linked-list-insert-delete|单链表的插入与删除]]。
