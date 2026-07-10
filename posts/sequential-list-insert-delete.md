---
title: Sequential List Insert Delete
tags:
  - DataStructureAndAlgorithm
  - LinearList
categories:
  - Data Structure & Algorithm
date: 2026-06-27 13:13:25
katex: true
---

# 顺序表的插入与删除

## 插入操作

`ListInsert(&L, i, e)` 表示在顺序表 `L` 的第 `i` 个位置插入元素 `e`。

[html-card](../assets/sequential-list-insert.html)

合法范围：`1 <= i <= L.length + 1`。

基本步骤：

1. 判断插入位置是否合法。
2. 判断表是否已满。
3. 从最后一个元素开始，到第 `i` 个元素为止，依次后移一位。
4. 将 `e` 放入第 `i` 个位置，即数组下标 `i - 1`。
5. `length` 加 1。

注意移动顺序必须从后往前，否则会覆盖尚未移动的元素。

```c
bool ListInsert(SqList *list, int index, ElemType value) {
    if (index < 1 || index > list->length + 1) return false;
    if (list->length >= MaxSize) return false;

    for (int moveIndex = list->length; moveIndex >= index; moveIndex--) {
        list->data[moveIndex] = list->data[moveIndex - 1];
    }
    list->data[index - 1] = value;
    list->length++;
    return true;
}
```

代码中的 `moveIndex` 是数组下标相关的位置。插入第 `index` 位时，原第 `index` 位到原表尾都要右移一格，所以循环从 `list->length` 开始，目标位置是 `data[moveIndex]`，来源位置是 `data[moveIndex - 1]`。

## 插入时间复杂度

设表长为 `n`。

- 最好情况：插入表尾，不需要移动元素，`O(1)`。
- 最坏情况：插入表头，需要移动 `n` 个元素，`O(n)`。
- 平均情况：插入位置等概率出现，移动期望为 `n/2` 个元素，`O(n)`。

## 删除操作

`ListDelete(&L, i, &e)` 表示删除第 `i` 个位置的元素，并用 `e` 返回被删除元素。

[html-card](../assets/sequential-list-delete.html)

合法范围：`1 <= i <= L.length`。

基本步骤：

1. 判断删除位置是否合法。
2. 用 `e` 保存第 `i` 个元素的值。
3. 从第 `i + 1` 个元素开始，依次前移一位。
4. `length` 减 1。

```c
bool ListDelete(SqList *list, int index, ElemType *deletedValue) {
    if (index < 1 || index > list->length) return false;

    *deletedValue = list->data[index - 1];
    for (int moveIndex = index; moveIndex < list->length; moveIndex++) {
        list->data[moveIndex - 1] = list->data[moveIndex];
    }
    list->length--;
    return true;
}
```

删除第 `index` 位后，原第 `index + 1` 位要补到第 `index` 位，所以循环从数组下标 `index` 对应的元素开始，把 `data[moveIndex]` 复制到 `data[moveIndex - 1]`。

## 删除时间复杂度

设表长为 `n`。

- 最好情况：删除表尾，不需要移动元素，`O(1)`。
- 最坏情况：删除表头，需要移动 `n - 1` 个元素，`O(n)`。
- 平均情况：删除位置等概率出现，平均移动约 `(n - 1)/2` 个元素，`O(n)`。

## 易错点

- 位序从 1 开始，数组下标从 0 开始。
- 插入第 `i` 个位置时，元素放入 `data[i - 1]`。
- 删除时若不使用 `&e`(取址)，被删除值无法带回调用处。
- 顺序表插入删除的主要时间开销来自移动元素。
- 插入必须从后往前移动；删除必须从前往后移动。方向错了就会覆盖仍需保留的数据。

## 关联

链式存储的插入删除见 [[singly-linked-list-insert-delete|单链表的插入与删除]]。顺序表插入删除的主要成本是移动元素，链表的主要成本是定位前驱结点。
