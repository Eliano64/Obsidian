---
title: KMP Nextval
tags:
  - String
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:34:00
katex: true
---

# KMP 的 nextval 优化

[[kmp-boundary-next|next 数组]]已经避免了主串指针回退，但仍可能产生无效比较。

## 为什么需要 nextval

若 `next[j] = k` 且 `T[j] == T[k]`，那么当 `T[j]` 与当前主串字符失配时，当前主串字符一定也不等于 `T[k]`。此时再退到 `k` 比较一次，必然再次失配。

因此可以直接跳到 `next[k]`，跳过这次必败比较。

## 递推关系

常见写法：

```c
void GetNextval(SString t, int next[], int nextval[]) {
    nextval[1] = 0;
    for (int j = 2; j <= t.length; ++j) {
        int k = next[j];
        if (k != 0 && t.ch[j] == t.ch[k]) {
            nextval[j] = nextval[k];
        } else {
            nextval[j] = k;
        }
    }
}
```

更口语化地说：

- 若退到的位置字符和失配字符相同，就继续沿着 `next` 退。
- 若退到的位置字符不同，保留该退回位置。

## 例 1：`ababaa`

| j | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---:|---:|---:|---:|---:|---:|
| T[j] | a | b | a | b | a | a |
| next[j] | 0 | 1 | 1 | 2 | 3 | 4 |
| nextval[j] | 0 | 1 | 0 | 1 | 0 | 4 |

解释：

- `j=3` 时，`T[3]=a`，`next[3]=1`，且 `T[1]=a`，退到 1 必败，所以 `nextval[3]=nextval[1]=0`。
- `j=6` 时，`T[6]=a`，`next[6]=4`，`T[4]=b`，不同，保留 4。

## 例 2：`abaabc`

`abaabc` 是手算 `next` 时常用的模式串：

| j | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---:|---:|---:|---:|---:|---:|
| T[j] | a | b | a | a | b | c |
| next[j] | 0 | 1 | 1 | 2 | 2 | 3 |
| nextval[j] | 0 | 1 | 0 | 2 | 1 | 3 |

解释：

- `j=3` 时，`T[3]=a`，`next[3]=1`，`T[1]=a`，退到 1 必败，所以 `nextval[3]=0`。
- `j=5` 时，`T[5]=b`，`next[5]=2`，`T[2]=b`，退到 2 必败，所以继续退到 `nextval[2]=1`。
- `j=4`、`j=6` 退回位置的字符不同，保留原 `next` 值。

## 例 3：`aaaab`

| j | 1 | 2 | 3 | 4 | 5 |
|---|---:|---:|---:|---:|---:|
| T[j] | a | a | a | a | b |
| next[j] | 0 | 1 | 2 | 3 | 4 |
| nextval[j] | 0 | 0 | 0 | 0 | 4 |

连续相同字符会让普通 `next` 产生多次必败回退，`nextval` 会把这些回退压缩掉。

## 使用

KMP 主体不变，只把失配时的回退数组从 `next` 换成 `nextval`：

```c
if (j == 0 || s.ch[i] == t.ch[j]) {
    ++i;
    ++j;
} else {
    j = nextval[j];
}
```

`nextval` 优化的是常数级比较次数，不改变 KMP 的总体时间复杂度 $O(n+m)$。
