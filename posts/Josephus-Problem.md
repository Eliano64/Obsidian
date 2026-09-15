---
title: Josephus Problem
tags:
  - Mathematics
categories:
  - Data Structure & Algorithm
date: 2026-09-15 17:10:19
katex: true
---
# 问题

$n$ 个人围成一个圈，编号依次为 $0,1,\dots,n-1$。从编号 0 的人开始按 $1,2,3,\dots$ 报数，报到 $m$ 的人出列；然后从出列者的下一个人重新从 1 报数，报到 $m$ 的人再出列。如此继续，求最后剩下的人的编号。

若题目要求编号从 1 开始，把下面各解法的结果加 1 即可。

# 解法一：递推

## 推导

记 $n$ 个人、报数到 $m$ 出列时的幸存者编号为 $f(n,m)$。

基线：只剩一个人时他一定是幸存者，所以 $f(1,m)=0$。

递推：$n$ 个人时第一个出列的是编号 $m-1$ 的人。他出列后剩 $n-1$ 个人，从编号 $m$ 的人开始重新报数，这正是一个规模为 $n-1$ 的同类问题。把剩下的人重新编号：

$$
\{0,1,\dots,m-2,m,\dots,n-1\}=\{m,m+1,\dots,n-1,0,1,\dots,m-2\}=\{(x+m)\%n\mid x=0,1,\dots,n-2\}
$$

即原问题剩下的 $n-1$ 个人，恰好是子问题中编号为 $x$ 的人整体平移 $m$ 位。子问题与原问题结构相同，解也满足同样的关系：

$$
f(n,m)=\bigl(f(n-1,m)+m\bigr)\%n
$$

## 代码

```cpp
int Josephus(int n, int m)
{
    if (n == 1)
    {
        return 0;                          // 只剩一人，编号为 0
    }
    return (Josephus(n - 1, m) + m) % n;    // 把子问题的解平移回原问题的编号
}
```

时间复杂度 $O(n)$，但递归深度为 $n$，可以改写成迭代，避免递归栈：

```cpp
int JosephusIter(int n, int m)
{
    int ans = 0;
    for (int i = 2; i <= n; i++)
    {
        ans = (ans + m) % i;                // i 个人时的幸存者编号
    }
    return ans;
}
```

# 解法二：模拟

每轮把出列的人从待选集合中删除，最后剩下的就是答案。递推解法依赖于"最后一个人"这一特定目标，而模拟法容易推广，例如把问题改成求最后 $k$ 个剩下的人的编号。

## 出列者在当前队列中的位置

用数组下标表示时：设某一轮开始时还剩 `remain` 个人，`vic` 是这一轮出列者在当前队列中的下标。第一个出列者的下标是

```cpp
int vic = (m - 1) % remain;
```

出列者被删除后，原本位于 `vic + 1` 的人移到了下标 `vic` 处，也就是下一轮的报数起点，于是

```cpp
vic = (vic + m - 1) % remain;   // remain 已更新为本轮删除后的人数
```

用链表表示时不需要维护下标：让指针 `prev` 始终停在报数起点之前，每轮从它出发走 `(m - 1) % remain` 步，就正好落在这一轮出列者的前一个结点上。

## 用 vector.erase 实现

```cpp
int Josephus(int m, vector<int>& candidate)
// candidate[i] == i
{
    int vic = (m - 1) % candidate.size();
    while (candidate.size() > 1)
    {
        candidate.erase(candidate.begin() + vic);   // 移除出列的人
        vic = (vic + m - 1) % candidate.size();     // 下一轮出列者在更新后队列中的下标
    }
    return candidate[0];
}
```

`vector.erase(iterator)` 删除一个元素要移动其后的所有元素，代价为 $O(n_1)$，$n_1$ 是当前剩下的人数。于是总时间复杂度为 $O(n^2)$。

## 用循环链表实现

```cpp

// struct node
//{
//    int index;
//    node* next;
//    node(int x) : index(x), next(NULL) {}
// };

// prev 指向本轮报数起点之前的那一个结点，这样便于删除出列的结点
int Josephus(int m, node* prev, int num)
{
    int step = (m - 1) % num;           // 从 prev 走到出列者前一结点的步数
    while (num > 1)
    {
        for (int i = 0; i < step; i++)
        {
            prev = prev->next;
        }
        prev->next = prev->next->next;   // 移除出列的人
        num--;
        step = (m - 1) % num;            // prev 恰好落在新起点之前，按新人数重算步数
    }
    return prev->index;
}

int main()
{
    int n, m;
    cin >> n >> m;
    node* head = NULL;
    node* prev = NULL;
    for (int i = 0; i < n; i++)
    {
        node* candidate = new node(i);
        if (head == NULL)
        {
            head = candidate;
        }
        else
        {
            prev->next = candidate;
        }
        prev = candidate;
    }
    prev->next = head;                  // 首尾相接，构成循环链表
    cout << Josephus(m, prev, n);
}
```

链表解法省去了删除元素时的整体搬移，但每轮仍要沿环走 `vic` 步，最坏情况下总步数为 $O(nm)$，因此时间复杂度为 $O(nm)$。
