---
title: Recursion And Stack
tags:
  - DataStructureAndAlgorithm
  - Stack
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:12:00
katex: true
---

# 递归与栈

递归适合处理“原问题可以转化为同类但规模更小的问题”的场景。递归定义必须同时包含递归体和递归出口，否则调用会无限深入。

## 递归工作栈

函数递归调用时，系统会维护函数调用栈，也称递归工作栈。每进入一层递归，就把该层调用所需的信息压入栈顶；每返回一层，就从栈顶弹出对应信息。

![Recursion working stack](../assets/recursion-working-stack.svg)

一层递归调用中通常需要保存：

- 参数值。
- 局部变量。
- 返回地址。
- 调用现场中还需要恢复的信息。

## 阶乘递归

```c
int Factorial(int n) {
    if (n == 0) {
        return 1;
    }
    return n * Factorial(n - 1);
}
```

[html-card height=900 step=80](../assets/factorial-recursion-stack.html)

递归表达式：

$$
factorial(n)=
\begin{cases}
n \times factorial(n-1), & n>0 \\
1, & n=0
\end{cases}
$$

调用 `Factorial(5)` 时，会依次压入 `n=5,4,3,2,1,0` 的调用记录；到达出口后再按相反顺序返回。这正体现了 [[stack-definition-and-operations|栈]] 的后进先出。

## 递归的代价

递归的主要代价来自两部分：

- 时间代价：函数调用本身有额外开销，有些递归还会重复计算。
- 空间代价：递归深度越大，调用栈占用越多，过深时可能栈溢出。

若每层递归只占常量空间，递归深度为 `n`，空间复杂度通常是 `O(n)`。若每层还申请与当前规模有关的数组，空间复杂度要把各层空间累加。

## 斐波那契递归的重复计算

```c
int Fibonacci(int n) {
    if (n <= 1) {
        return n;
    }
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}
```

朴素斐波那契递归会反复计算相同子问题，例如 `Fibonacci(n-2)` 会在多个分支中重复出现。它能体现递归思想，但效率低，不适合作为高效实现。

## 递归转非递归

递归算法可以用显式栈改写成非递归算法。详见[[../../../posts/recursion-to-loop|recursion-to-loop]]

改写时要明确：

1. 每个栈帧需要保存哪些变量。
2. 进入下一层前压入什么信息。
3. 返回上一层时如何恢复现场。
4. 递归出口对应非递归循环中的哪个终止条件。
