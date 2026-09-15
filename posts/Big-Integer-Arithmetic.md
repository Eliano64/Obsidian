---
title: Big Integer Arithmetic
tags:
  - Mathematics
categories:
  - Data Structure & Algorithm
date: 2026-09-15 17:10:19
katex: true
---

# 使用 `string` 来表示数

内置整数类型的取值范围有上限，例如 `unsigned long long int` 最大只能表示 $2^{64}-1$。当参与运算的整数超出这个范围时，就把它的十进制数字串存进 `string`，逐位模拟进位，手工完成运算。

本文只涉及加法和乘法。

两个待处理的数分别记为 `big` 和 `small`，二者的位数分别为 $L_1$ 和 $L_2$。

# 加法

## 模拟过程

用 `result` 保存结果，初始为空。

从最低位开始，把两个数的当前位与低位送来的进位相加，得到 $num=10a+b$：$a$ 是送给上一位的进位，$b$ 是本位结果，把 $b$ 追加到 `result` 的前面。

为了方便对齐，先把位数较少的数在左侧补 `"0"`，使两个数等长。所有位处理后如果仍有进位，再把它加到 `result` 前面。

## 代码

```cpp
string add(string big, string small)
{
    string result;
    if (small.size() > big.size())
    {
        swap(big, small);           // 让 big 保存位数较多的数
    }
    int l_len = big.size();
    int s_len = small.size();
    for (int i = 0; i < l_len - s_len; i++)
    {
        small = '0' + small;        // 左侧补零，使两数等长
    }

    int up = 0;                     // 进位
    for (int i = l_len - 1; i >= 0; i--)
    {
        int num = (big[i] - '0') + (small[i] - '0') + up;
        up = num / 10;              // 本位向高位的进位
        num %= 10;                  // 本位结果
        result = (char)(num + '0') + result;
    }
    if (up != 0)
    {
        result = (char)(up + '0') + result;   // 最高位仍有进位
    }

    return result;
}
```

## 复杂度

每一位只处理一次，时间复杂度 $O(L_1)$，其中 $L_1$ 是较长那个数的位数。

# 乘法

## 模拟过程

用 `result` 保存结果，初始为空。

把位数较少的数记为 `small`，从它的最低位开始，逐位与位数较多的数 `big` 相乘。对 `small` 的每一位：

用 `part` 保存这一位的部分积，初始为空。`big` 从低位到高位依次与 `small` 的这一位相乘，同时加上低位送来的进位，得到 $num=10a+b$：$a$ 是送给高位的进位，$b$ 是本位结果，把 $b$ 追加到 `part` 前面。`big` 的所有位处理完，如果仍有进位，把它加到 `part` 前面。

这一位在 `small` 中位于从低位数起的第 $k$ 位，就把 `part` 右端补上 $k$ 个 `"0"`，相当于乘 $10^k$。最后累加各次的部分积：

```cpp
result = add(result, part);
```

`small` 的每一位都处理完，`result` 即为乘积。

## 代码

```cpp
string mul(string big, string small)
{
    string result;
    if (small.size() > big.size())
    {
        swap(big, small);           // 让 big 保存位数较多的数
    }
    int l_len = big.size();
    int s_len = small.size();

    for (int i = s_len - 1; i >= 0; i--)   // 从 small 的最低位开始
    {
        int up = 0;                        // 进位
        string part = "";
        for (int j = l_len - 1; j >= 0; j--)
        {
            int num = (big[j] - '0') * (small[i] - '0') + up;
            up = num / 10;
            num %= 10;
            part = (char)(num + '0') + part;
        }
        if (up != 0)
        {
            part = (char)(up + '0') + part;    // 最高位仍有进位
        }
        for (int k = 0; k < s_len - 1 - i; k++)
        {
            part += '0';                       // 这一位在 small 中的位权
        }
        result = add(result, part);            // 累加部分积
    }

    return result;
}
```

## 复杂度

`small` 的每一位都要与 `big` 的所有位相乘，再做一次位数约为 $O(L_1+L_2)$ 的加法，因此时间复杂度为 $O(L_1L_2)$。

# 推广

- 小数的高精度计算：整数部分照上面处理，最后单独确定小数点的位置即可。
- $n$ 进制的大数运算：把代码里的 10 与 `'0'` 换成对应的进制基数与字符起点即可；加法的改动尤其小。
