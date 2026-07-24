---
title: Backtracking
tags:
  - Search
categories:
  - Data Structure & Algorithm
date: 2026-06-24 00:00:00
katex: true
---

## 回溯法的本质

**回溯法**是一种系统枚举解空间的方法。它把求解过程看成一棵搜索树：

- 每一层对应一个决策阶段；
- 每条边对应一种选择；
- 从根到叶的一条路径对应一个候选解；
- 如果当前路径已经不可能得到合法解或更优解，就提前停止继续向下搜索。

回溯的核心动作是：

1. **选择**：在当前状态下尝试一种可能。
2. **递归**：进入下一层继续搜索。
3. **撤销选择**：递归返回后恢复现场，让同层的其他选择能够在干净状态下继续尝试。

因此，回溯不是单纯的递归。它强调“递归前改变状态，递归后恢复状态”。

[html-card height=1080](../assets/backtracking-n-queens-process.html)

## 适用条件

当问题满足下面特征时，通常可以考虑回溯：

| 条件 | 含义 |
|---|---|
| 分步决策 | 答案可以由若干步选择组成 |
| 层次结构明显 | 第 $k$ 步的选择依赖前 $k-1$ 步的状态 |
| 需要枚举 | 需要找全部方案、某个可行方案，或在全部可行方案中找最优方案 |
| 可剪枝 | 可以在中途判断某些分支一定无效或不优 |

常见题型包括 N 皇后、全排列、组合、子集、迷宫路径、数独、0-1 背包搜索版、图中的路径枚举等。

暴力递归如果不剪枝，搜索树可能指数级增长。使用回溯前要先估计状态规模，否则容易超时。

## 通用模板

```c
void backtrack(int step) {
    if (step == target_step) {
        /* 当前路径已经形成一个完整候选解。 */
        record_answer();
        return;
    }

    for (int choice = first_choice(step); choice <= last_choice(step); choice++) {
        if (!is_valid(step, choice)) {
            continue;  /* 可行性剪枝：非法选择不再向下搜索。 */
        }

        make_choice(step, choice);       /* 选择：修改当前状态。 */
        backtrack(step + 1);             /* 递归：进入下一层。 */
        undo_choice(step, choice);       /* 回溯：撤销本层选择。 */
    }
}
```

模板中的关键不是函数名，而是状态变化的对称性：

```c
make_choice(...);
backtrack(...);
undo_choice(...);
```

如果漏掉 `undo_choice`，递归返回后，上一层状态会残留下一层的修改，同层其他分支就会被污染。

## 组合问题中的 start

组合问题不关心元素顺序。例如 `{1, 3}` 和 `{3, 1}` 是同一个组合。为了避免重复枚举，通常约定元素只能按下标递增顺序选取。

```c
void combine(int start, int depth) {
    if (depth == k) {
        record_answer();
        return;
    }

    for (int i = start; i < n; i++) {
        path[depth] = a[i];
        combine(i + 1, depth + 1);
    }
}
```

这里的 `start` 表示“本层只能从哪个下标开始选”。递归时传入 `i + 1`，就能保证后面不会再选到当前元素以及它前面的元素，从而自然排除重复排列。

## N 皇后问题

N 皇后要求在 $n \times n$ 的棋盘上放置 $n$ 个皇后，使任意两个皇后都不在同一行、同一列、同一条对角线上。

### 状态设计

按“行”逐层放置皇后：

- `row` 表示当前要放第几行；
- 每一行只放一个皇后，因此本层只需要枚举列 `col`；
- `col_used[col]` 记录某列是否已有皇后；
- `diag1[row + col]` 记录主对角线是否被占用；
- `diag2[row - col + n - 1]` 记录副对角线是否被占用。

两个对角线编号的依据：

| 对角线   | 同一对角线上的不变量     |
| ----- | -------------- |
| 主对角线  | `row - col` 相同 |
| 副对角线  | `row + col` 相同 |

因为 `row - col` 可能为负，所以常用 `row - col + n - 1` 平移成非负下标。

### C 实现

```c
#include <stdio.h>
#include <stdbool.h>

#define MAX_N 20

static int n;
static int solution_count;
static int queen_col[MAX_N];
static bool col_used[MAX_N];
static bool diag_sum_used[2 * MAX_N];
static bool diag_diff_used[2 * MAX_N];

static void print_solution(void) {
    for (int row = 0; row < n; row++) {
        for (int col = 0; col < n; col++) {
            putchar(queen_col[row] == col ? 'Q' : '.');
        }
        putchar('\n');
    }
    putchar('\n');
}

static bool can_place(int row, int col) {
    int diag_sum = row + col;
    int diag_diff = row - col + n - 1;

    return !col_used[col] &&
           !diag_sum_used[diag_sum] &&
           !diag_diff_used[diag_diff];
}

static void place_queen(int row, int col) {
    int diag_sum = row + col;
    int diag_diff = row - col + n - 1;

    queen_col[row] = col;
    col_used[col] = true;
    diag_sum_used[diag_sum] = true;
    diag_diff_used[diag_diff] = true;
}

static void remove_queen(int row, int col) {
    int diag_sum = row + col;
    int diag_diff = row - col + n - 1;

    queen_col[row] = -1;
    col_used[col] = false;
    diag_sum_used[diag_sum] = false;
    diag_diff_used[diag_diff] = false;
}

static void solve_n_queens(int row) {
    if (row == n) {
        solution_count++;
        print_solution();
        return;
    }

    for (int col = 0; col < n; col++) {
        if (!can_place(row, col)) {
            continue;
        }

        place_queen(row, col);
        solve_n_queens(row + 1);
        remove_queen(row, col);  /* 恢复现场，继续尝试本行的下一列。 */
    }
}

int main(void) {
    if (scanf("%d", &n) != 1 || n <= 0 || n > MAX_N) {
        return 0;
    }

    for (int i = 0; i < n; i++) {
        queen_col[i] = -1;
    }

    solve_n_queens(0);
    printf("%d\n", solution_count);
    return 0;
}
```


## 剪枝

**剪枝**是在搜索树中提前砍掉“不可能产生合法答案”或“不可能产生更优答案”的分支。剪枝不会改变问题的解空间，只是避免访问没有必要访问的状态。

| 类型 | 含义 | 例子 |
|---|---|---|
| 可行性剪枝 | 当前状态已经不可能合法 | N 皇后中同列或同斜线已有皇后 |
| 最优性剪枝 | 当前状态继续搜索也不可能超过已有答案 | 当前收益 + 剩余理论上界不超过当前最优值 |
| 边界剪枝 | 越界、资源不足、步数超限 | 迷宫坐标越界、容量为负 |
| 去重剪枝 | 跳过等价分支 | 排序后跳过同层重复元素 |
| 顺序剪枝 | 先搜索更可能成功或更优的分支 | 先放约束最多的位置 |

剪枝的前提是正确性：被剪掉的分支必须确定不会影响答案。

## 易错点

| 易错点 | 后果 |
|---|---|
| 只递归，不撤销选择 | 状态污染，同层其他分支判断错误 |
| 递归终止条件写晚 | 已经形成完整解后仍继续向下枚举 |
| 组合问题没有使用 `start` | 同一个组合被按不同顺序重复统计 |
| 剪枝条件过强 | 合法答案被误删 |
| 只估计单条路径深度 | 忽略整棵搜索树规模，导致复杂度误判 |
