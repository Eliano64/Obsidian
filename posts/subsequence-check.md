---
title: subsequence check
tags:
  - String
  - DataStructureAndAlgorithm
categories:
  - Data Structure & Algorithm
date: 2025-09-26 00:00:00
katex: true
---

# 1. 介绍

引例：
有一个字符串t。现有若干字符集合$s_i$，需要判断对于每一个$s_i$，它的每一个元素是否在t中出现。

一种方式是，对于t中的所有字符建立一个map。然后对于一个$s_i$中的每一个元素c，若`map[c]`都判断为存在，则$s_i$的每一个元素都在t中出现，否则不是。

现在将问题换为“有一个字符串t。现有一个字符串集合$s$，需要判断对于每一个元素$e_i$，它是否是t的**子序列**。”

这个时候，不仅仅要考虑字符的存在与否了，还要考虑字符之间的相对位置关系。

比方说，`t=aaabbb`，`e=ba`。判断e是否是t的子序列时，我们还要判断在t的`b`后面的元素中是否还有`a`。

我们使用的`map`得能够“屏蔽”在t的某个元素前面的所有元素的存在性。

于是使用`nxt`，`nxt[i][c]`: t中下标不小于`i`的元素中出现`c`的最小下标。

# 2. 解释

将`nxt[t.length][c]`初始化一个很大的值`BIG`，作为判断存在依据。

其他`nxt[i][c]`构建：

```txt
for i in t.length-1 to 0; do
    copy nxt[i+1] to nxt[i]
    nxt[i][t[i]] = i
end loop
```

对于一个字符串$e_i$判断:

```txt
idx := 0
for c in e_i; do
    idx = nxt[idx][c]
    if idx == BIG; then
        c does not exist in t[idx:]=> e_i is not a substring of t.
    else 
        idx++ # for next search should begin at idx+1
    fi
end loop
```

# 3. 例题

[匹配子序列的单词数](https://leetcode.cn/problems/number-of-matching-subsequences)

给出go的解法示例

```go
func numMatchingSubseq(s string, words []string) int {
    nxt := make([][26]int,len(s)+1)
    for i := range nxt[len(s)]{
        nxt[len(s)][i] = 2147483647
    }

    for i:=len(s)-1;i>=0;i--{
        nxt[i] = nxt[i+1] //nxt是数组的slice,nxt[i]是数组类型。数组是值类型，可以直接这样赋值
        nxt[i][s[i]-'a']=i
    }

    ans := 0
    for _,str := range words{
        idx := 0
        for _,c := range str{
            idx = nxt[idx][c-'a']
            if(idx==2147483647){
                ans--
                break
            }
            idx++
        }
        ans++
    }
    return ans
}
```
