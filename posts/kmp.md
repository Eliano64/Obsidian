---
title: KMP
tags:
  - String
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:33:00
katex: true
---

# KMP

KMP 解决的问题与[[naive-pattern-matching|朴素模式匹配]]相同：在主串中查找模式串第一次出现的位置。区别在于：KMP 发生失配时，主串指针 `i` 不回退，只根据模式串自身结构调整模式串指针 `j`。

# 核心思想


当 `S[i] != T[j]` 时，`T[1..j-1]` 已经与主串中的一段字符完全相同。KMP 利用这段“已知匹配信息”，判断模式串应该滑到哪里继续比较。

`next[j]` 表示：当 `T[j]` 与 `S[i]` 失配，且 `T[1..j-1]` 已经匹配时，模式串指针 `j` 应退到的位置。

- `next[1] = 0`：第一个字符失配，说明当前主串字符无法匹配模式串开头；下一步应移动主串指针。
- `next[2] = 1`：第二个字符失配时，尝试让模式串第一个字符与当前主串字符比较。
- 其他 `next[j]` 的计算方法：

	1. 在失配位置前画一条分界线。
	2. 只看分界线之前已经匹配的部分。
	3. 让模式串一步一步后退，直到分界线之前模式串部分能与相应位置的主串部分还能对上，或分界线前的模式串部分为$\emptyset$。**至少后退一次。**
	4. 此时 `j` 指向的位置，就是 `next[j]`。

> [! example] T = abaabc, 计算 `next[6]` (约定数组序号从 1 开始)。
>```text
> abaab | 
> abaab | c  j = 6 
>  abaa | b  j = 5  (abaa != baab)
>   aba | a  j = 4  (aba != aab)
>    ab | a  j = 3 (ab == ab, success!)
> ```

[html-card height=800 step=80](../assets/kmp-boundary-next.html)

# 代码实现

## `next` 数组计算

```C

// 模式串退后 step 步时, 是否能与 idx 之前的部分的相应位置的部分对上
int compare(SString s, int idx, int step){
	for(int i = 0;i+step < idx;++i){
		if(s[i] != s[i+step]){
			return false;
		}
	}
	return true;
}

void GetNext(SString t, int next[]){
	next[1] = 0;
	next[2] = 1;
    for(int i = 3;i <= t.length; ++i){
		for(int j = 1;j <= i; ++j){
			if(compare(t,i,j)){
				next[i] = i-j;
				break;
			}
		}
	}
}

```

## KMP 匹配

```c
int KMP_Index(SString s, SString t, int next[]) {
    int i = 1;  // 主串 s 的当前比较位置，采用 1-based 下标
    int j = 1;  // 模式串 t 的当前比较位置

    while (i <= s.length && j <= t.length) {
        if (j == 0 || s.ch[i] == t.ch[j]) {
            // j == 0 表示 t[1] 也无法匹配当前 s[i]，
            // 需要放弃当前主串字符，让 i 前进，并把 j 恢复到 1。
            // 字符相等时，说明当前一对字符匹配成功，也同时前进。
            ++i;
            ++j;
        } else {
            // t[j] 失配，但 t[1..j-1] 已匹配。
            // next[j] 给出模式串可保留的前缀之后，下一次应比较的位置。
            // 这里主串指针 i 不回退，避免重复比较已经确认匹配的字符。
            j = next[j];
        }
    }

    if (j > t.length) {
        // 匹配成功时，i 已经指向匹配段后一个位置。
        // 因此起始位置 = i - 模式串长度。
        return i - t.length;
    }

    // 主串扫描完仍未让 j 越过模式串末尾，表示匹配失败。
    return 0;
}
```

`j == 0` 的含义：模式串第一个字符也无法匹配当前主串字符，需要让主串指针前进，并让 `j` 回到 1。

# 复杂度

对于空间复杂度，保存 `next` 数组需要 $O(m)$ 的额外空间。

对于时间复杂度，设主串长度为 $n$，模式串长度为 $m$。KMP 的时间包括构造 `next` 和匹配两部分。

构造 `next` 需要遍历整个模式串，因此为 $O(m)$。

在匹配阶段，**时间由主串指针 `i` 的移动决定，而主串指针 `i` 只向后移动，不会因失配而回退。** 因此，主串最多被从头到尾扫描一遍，所以匹配阶段时间复杂度最坏为 $O(n)$。

因此，KMP 的时间复杂度为：

$$
O(m)+O(n)=O(m+n)
$$

# 优化

若 `next[j] = k` 且 `T[j] == T[k]`，那么当 `T[j]` 与当前主串字符失配时，当前主串字符一定也不等于 `T[k]`。此时再退到 `k` 比较一次，必然再次失配。

因此可以直接跳到 `next[k]`，跳过这次必失配的比较。称这个优化后的 `next` 数组为 `nextval`。

以下代码展示在 `next` 数组计算出来后再进行优化的写法。当然可以在计算 `next` 的同时就进行优化。

```c
void GetNextval(SString t, int next[], int nextval[]) {
    nextval[1] = 0;
    for (int j = 2; j <= t.length; ++j) {
        int k = next[j];
        if (k != 0 && t.ch[j] == t.ch[k]) {
            nextval[j] = nextval[k]; // 若退到的位置字符和失配字符相同，就继续沿着 `next` 退。
        } else {
            nextval[j] = k; //若退到的位置字符不同，保留该退回位置。
        }
    }
}
```

`nextval` 在 `kmp` 算法中使用方式与 `next` 数组一样。

它优化的是常数级比较次数，不改变 KMP 的总体时间复杂度 $O(n+m)$。

> [! exmaple]
> 1. 例 1：`ababaa`
> 
> | j | 1 | 2 | 3 | 4 | 5 | 6 |
> |---|---:|---:|---:|---:|---:|---:|
> | T[j] | a | b | a | b | a | a |
> | next[j] | 0 | 1 | 1 | 2 | 3 | 4 |
> | nextval[j] | 0 | 1 | 0 | 1 | 0 | 4 |
>
>
> 
> - `j=3` 时，`T[3]=a`，`next[3]=1`，且 `T[1]=a`，退到 1 必然失配，所以 `nextval[3]=nextval[1]=0`。
> - `j=6` 时，`T[6]=a`，`next[6]=4`，`T[4]=b`，不同，保留 4。
> 
> 2. 例 2：`abaabc`
> 
> | j | 1 | 2 | 3 | 4 | 5 | 6 |
> |---|---:|---:|---:|---:|---:|---:|
> | T[j] | a | b | a | a | b | c |
> | next[j] | 0 | 1 | 1 | 2 | 2 | 3 |
> | nextval[j] | 0 | 1 | 0 | 2 | 1 | 3 |
> 
>
> 
> - `j=3` 时，`T[3]=a`，`next[3]=1`，`T[1]=a`，退到 1 必然失配，所以 `nextval[3]=0`。
> - `j=5` 时，`T[5]=b`，`next[5]=2`，`T[2]=b`，退到 2 必然失配，所以继续退到 `nextval[2]=1`。
> - `j=4`、`j=6` 退回位置的字符不同，保留原 `next` 值。
> 
> 
> 3. 例 3：`aaaab`
> 
> | j | 1 | 2 | 3 | 4 | 5 |
> |---|---:|---:|---:|---:|---:|
> | T[j] | a | a | a | a | b |
> | next[j] | 0 | 1 | 2 | 3 | 4 |
> | nextval[j] | 0 | 0 | 0 | 0 | 4 |
> 
> 连续相同字符会让普通 `next` 产生多次冗余回退，`nextval` 会把这些回退压缩掉。

