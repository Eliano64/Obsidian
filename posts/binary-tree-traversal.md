---
title: Binary Tree Traversal 
tags:
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:40:00 
katex: true
---

# 二叉树遍历

遍历是按照某种次序把二叉树中所有结点访问一遍，且每个结点只访问一次。

[html-card height=840 step=80](../assets/binary-tree-traversal.html)

# 三种递归遍历

三种遍历只改变“访问根”的时机，对于左右子节点都是先左后右：

| 遍历 | 缩写 | 次序 |
|---|---|---|
| 先序遍历 | NLR | 根、左、右 |
| 中序遍历 | LNR | 左、根、右 |
| 后序遍历 | LRN | 左、右、根 |


```c
void PreOrder(BiTree root) {
    if (root == NULL) return;
    visit(root);
    PreOrder(root->lchild);
    PreOrder(root->rchild);
}

void InOrder(BiTree root) {
    if (root == NULL) return;
    InOrder(root->lchild);
    visit(root);
    InOrder(root->rchild);
}

void PostOrder(BiTree root) {
    if (root == NULL) return;
    PostOrder(root->lchild);
    PostOrder(root->rchild);
    visit(root);
}
```

递归遍历的时间复杂度都是 $O(n)$，因为每个结点访问一次。递归调用栈的空间复杂度与树高有关，最坏为 $O(n)$，若树比较平衡则为 $O(\log_2 n)$。

# 层次遍历

层次遍历从上到下、从左到右访问结点，依赖[[Queue-Definition-And-Operations|队列]]。

```c
void LevelOrder(BiTree root) {
    if (root == NULL) return;

    InitQueue(&q);
    EnQueue(&q, root);

    while (!QueueEmpty(q)) {
        BiTNode *node;
        DeQueue(&q, &node);
        visit(node);

        if (node->lchild != NULL) EnQueue(&q, node->lchild);
        if (node->rchild != NULL) EnQueue(&q, node->rchild);
    }
}
```

# 递归遍历的非递归写法

递归遍历的递归写法依赖函数调用栈。非递归写法需要显式使用[[Stack-Definition-And-Operations|栈]]模拟函数调用过程。

更一般的递归转循环思路见[[Recursion-To-Loop-With-Stack]]。

对二叉树递归遍历来说，一个非空结点的递归函数大致有两个递归点：左子树、右子树。因此可以用 `ip` 表示当前栈帧执行到哪一步：

| `ip` | 含义 |
|---:|---|
| `0` | 还没进入左子树 |
| `1` | 左子树已经处理完，还没进入右子树 |
| `2` | 右子树已经处理完，当前结点即将结束 |


## 用栈帧模拟先序遍历

先序遍历是“根、左、右”，所以在 `ip == 0` 时访问当前结点，然后压入左子树。

```c
typedef struct {
    BiTNode *node;
    int ip;
} TraversalFrame;

void PreOrderByFrame(BiTree root) {
    TraversalFrame stack[128];
    int top = -1;

    stack[++top] = (TraversalFrame){root, 0};

    while (top != -1) {
        TraversalFrame *cur = &stack[top];

        if (cur->node == NULL) {
            --top;
            continue;
        }

        if (cur->ip == 0) {
            visit(cur->node);
            cur->ip = 1;
            stack[++top] = (TraversalFrame){cur->node->lchild, 0};
            continue;
        }

        if (cur->ip == 1) {
            cur->ip = 2;
            stack[++top] = (TraversalFrame){cur->node->rchild, 0};
            continue;
        }

        --top;
    }
}
```

## 用栈帧模拟中序遍历

中序遍历是“左、根、右”，所以访问当前结点的位置在左子树返回之后，也就是 `ip == 1` 时。

```c
void InOrderByFrame(BiTree root) {
    TraversalFrame stack[128];
    int top = -1;

    stack[++top] = (TraversalFrame){root, 0};

    while (top != -1) {
        TraversalFrame *cur = &stack[top];

        if (cur->node == NULL) {
            --top;
            continue;
        }

        if (cur->ip == 0) {
            cur->ip = 1;
            stack[++top] = (TraversalFrame){cur->node->lchild, 0};
            continue;
        }

        if (cur->ip == 1) {
            visit(cur->node);
            cur->ip = 2;
            stack[++top] = (TraversalFrame){cur->node->rchild, 0};
            continue;
        }

        --top;
    }
}
```

## 用栈帧模拟后序遍历

后序遍历是“左、右、根”，所以必须等左右子树都返回后才能访问当前结点，也就是 `ip == 2` 时。

```c
void PostOrderByFrame(BiTree root) {
    TraversalFrame stack[128];
    int top = -1;

    stack[++top] = (TraversalFrame){root, 0};

    while (top != -1) {
        TraversalFrame *cur = &stack[top];

        if (cur->node == NULL) {
            --top;
            continue;
        }

        if (cur->ip == 0) {
            cur->ip = 1;
            stack[++top] = (TraversalFrame){cur->node->lchild, 0};
            continue;
        }

        if (cur->ip == 1) {
            cur->ip = 2;
            stack[++top] = (TraversalFrame){cur->node->rchild, 0};
            continue;
        }

        visit(cur->node);
        --top;
    }
}
```

这三段代码的差异只在 `visit(cur->node)` 放在 `ip == 0`、`ip == 1` 还是 `ip == 2`。这正对应先序、中序、后序中“访问根”的时机。

# 由遍历序列构造二叉树

只知道一种遍历序列通常无法唯一确定二叉树。

常见可唯一确定的组合：
- 先序 + 中序。
- 后序 + 中序。
- 层序 + 中序。

不能唯一确定的组合：

- 先序 + 后序，通常不能唯一确定二叉树。但可以唯一确定层序遍历序列。
- 先序 + 后序 + 层序，通常也不能唯一确定二叉树，因为仍缺少中序序列给出的左右子树边界。

原因：中序序列能把左右子树分开。没有中序时，左右子树边界通常不确定。

> [!warning] 这些"不能唯一确定的组合"能唯一确定的情况
> 它们之所以不能唯一确定,本质是无法确定左右子树。那么,若树没有度为 1 的点, 则它们也可以唯一确定二叉树。

构造步骤：

- 先序 + 中序：先序第一个结点是根；用根在中序序列中划分左、右子树；再递归处理。
- 后序 + 中序：后序最后一个结点是根；用根在中序序列中划分左、右子树；再递归处理。
- 层序 + 中序：层序中第一个结点是根；用中序划分左右子树；再从层序序列中筛出属于左、右子树的结点，递归处理。

例如先序 `A D B C E`、中序 `B D C A E`：

1. 先序首元素 `A` 是根。
2. 中序中 `A` 左侧 `B D C` 是左子树，右侧 `E` 是右子树。
3. 左子树先序为 `D B C`，中序为 `B D C`，所以左子树根为 `D`。
4. `D` 的中序左侧为 `B`，右侧为 `C`，构造完成。

后序 `E F A H C I G B D`、中序 `E A F D H C B G I`：

1. 后序末元素 `D` 是根。
2. 中序中 `D` 左侧 `E A F` 是左子树，右侧 `H C B G I` 是右子树。
3. 左子树后序为 `E F A`，根为 `A`；右子树后序为 `H C I G B`，根为 `B`。
4. 继续按“后序末尾找根、中序划分左右子树”的规则递归。

层序 `D A B E F C G H I`、中序 `E A F D H C B G I`：

1. 层序首元素 `D` 是根。
2. 中序中 `D` 左侧 `E A F` 是左子树，右侧 `H C B G I` 是右子树。
3. 从剩余层序中筛出左子树结点，得到 `A E F`；筛出右子树结点，得到 `B C G H I`。
4. 左子树根为 `A`，右子树根为 `B`，再递归处理各子树。

层序 + 中序构造时要注意验证：每次划分后，层序序列中属于某一子树的结点必须全部能在该子树的中序范围内找到。
