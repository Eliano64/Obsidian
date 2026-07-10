---
title: Threaded Binary Tree 
tags:
  - DataStructureAndAlgorithm
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:41:00 
katex: true
---

# 线索二叉树

## 为什么需要线索

在含 `n` 个结点的二叉链表中，有 `n+1` 个空指针域，即 `n+1 个空指针域` 是可被利用的冗余链域。普通二叉链表只能方便地找到某结点的左右孩子；若要找遍历序列中的前驱或后继，通常还要重新遍历。

线索二叉树利用空指针域保存遍历序列中的前驱或后继。

## 结点结构

```c
typedef struct ThreadNode {
    ElemType data;
    struct ThreadNode *lchild;
    struct ThreadNode *rchild;
    int ltag;
    int rtag;
} ThreadNode, *ThreadTree;
```

标志位含义：

| 标志 | 值 | 含义 |
|---|---:|---|
| `ltag` | 0 | `lchild` 指向左孩子 |
| `ltag` | 1 | `lchild` 指向遍历前驱 |
| `rtag` | 0 | `rchild` 指向右孩子 |
| `rtag` | 1 | `rchild` 指向遍历后继 |

## 中序线索化

线索化时用 `pre` 记录刚访问过的结点。

```c
ThreadNode *pre = NULL;

void InThread(ThreadTree root) {
    if (root == NULL) return;

    InThread(root->lchild);

    if (root->lchild == NULL) {
        root->lchild = pre;
        root->ltag = 1;
    }
    if (pre != NULL && pre->rchild == NULL) {
        pre->rchild = root;
        pre->rtag = 1;
    }
    pre = root;

    InThread(root->rchild);
}
```

完整建立中序线索树时，还要处理遍历结束后的最后一个结点：

```c
void CreateInThread(ThreadTree root) {
    pre = NULL;
    if (root != NULL) {
        InThread(root);
        if (pre->rchild == NULL) {
            pre->rtag = 1;
        }
    }
}
```

最后一个结点没有中序后继，它的右线索应指向 `NULL`。

注意语句顺序：

1. 先处理左子树。
2. 当前结点左空则连前驱。
3. 前驱结点右空则连当前结点作为后继。
4. 更新 `pre`。
5. 再处理右子树。

## 先序线索化

先序线索化的访问次序是“根、左、右”。处理当前结点时，如果左指针为空就连前驱；如果前驱的右指针为空，就把前驱的右指针连向当前结点作为后继。

```c
void PreThread(ThreadTree root) {
    if (root == NULL) return;

    if (root->lchild == NULL) {
        root->lchild = pre;
        root->ltag = 1;
    }
    if (pre != NULL && pre->rchild == NULL) {
        pre->rchild = root;
        pre->rtag = 1;
    }
    pre = root;

    if (root->ltag == 0) {
        PreThread(root->lchild);
    }
    if (root->rtag == 0) {
        PreThread(root->rchild);
    }
}
```

先序线索化要特别注意：若当前结点的左指针已经被改成前驱线索，就不能再沿 `lchild` 递归，否则会把线索当成孩子访问。

## 后序线索化

后序线索化的访问次序是“左、右、根”。先处理左右子树，最后处理当前结点的前驱和后继线索。

```c
void PostThread(ThreadTree root) {
    if (root == NULL) return;

    PostThread(root->lchild);
    PostThread(root->rchild);

    if (root->lchild == NULL) {
        root->lchild = pre;
        root->ltag = 1;
    }
    if (pre != NULL && pre->rchild == NULL) {
        pre->rchild = root;
        pre->rtag = 1;
    }
    pre = root;
}
```

后序线索化本身不难，难点在后序线索树找后继，因为后序遍历最后才访问根，很多结点的后继与其双亲关系有关。

## 中序线索树找后继

```c
ThreadNode *FirstNode(ThreadNode *node) {
    while (node->ltag == 0) {
        node = node->lchild;
    }
    return node;
}

ThreadNode *NextNode(ThreadNode *node) {
    if (node->rtag == 1) {
        return node->rchild;
    }
    return FirstNode(node->rchild);
}
```

若 `rtag == 1`，右指针就是后继；若 `rtag == 0`，后继是右子树中最左下的结点。

## 中序线索树找前驱

```c
ThreadNode *LastNode(ThreadNode *node) {
    while (node->rtag == 0) {
        node = node->rchild;
    }
    return node;
}

ThreadNode *PreNode(ThreadNode *node) {
    if (node->ltag == 1) {
        return node->lchild;
    }
    return LastNode(node->lchild);
}
```

若 `ltag == 1`，左指针就是前驱；若 `ltag == 0`，前驱是左子树中最右下的结点。

## 三种线索二叉树的差异

| 类型 | 线索含义 | 容易直接找 | 不方便直接找 |
|---|---|---|---|
| 中序线索二叉树 | 中序前驱、中序后继 | 前驱、后继 | - |
| 先序线索二叉树 | 先序前驱、先序后继 | 后继 | 前驱 |
| 后序线索二叉树 | 后序前驱、后序后继 | 前驱 | 后继 |

先序线索树找先序后继：

- 若 `p` 有左孩子，先序后继是左孩子。
- 若 `p` 没有左孩子但有右孩子，先序后继是右孩子。
- 若相应指针为线索，则直接沿后继线索找到后继。

先序线索树找先序前驱通常不方便，因为先序是“根、左、右”，左右子树中的结点只可能是根的后继，不可能是根的前驱。若增加 `parent` 指针，则可分类讨论：`p` 是左孩子或父结点没有左孩子时，前驱为父结点；`p` 是右孩子且父结点有左子树时，前驱为左兄弟子树中最后一个被先序遍历的结点；根没有先序前驱。

后序线索树找后序前驱：

- 若 `p` 有右孩子，后序前驱是右孩子。
- 若 `p` 没有右孩子但有左孩子，后序前驱是左孩子。
- 若相应指针为线索，则直接沿前驱线索找到前驱。

后序线索树找后序后继通常不方便，因为后序是“左、右、根”，左右子树中的结点只可能是根的前驱，不可能是根的后继。若增加 `parent` 指针，则可分类讨论：`p` 是右孩子或父结点没有右孩子时，后继为父结点；`p` 是左孩子且父结点有右子树时，后继为右兄弟子树中第一个被后序遍历的结点；根没有后序后继。

线索二叉树的核心不是改变遍历次序，而是把某种遍历序列中的前驱、后继显式存到空链域里。
