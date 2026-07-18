---
title: String Definition And Operations
tags:
  - DataStructureAndAlgorithm
  - String
categories:
  - Data Structure & Algorithm
date: 2026-06-27 18:30:00
katex: true
---

# 串的定义与基本操作

## 串是什么

串，也叫字符串，是由零个或多个字符组成的有限序列，常记作：

$$
S='a_1a_2\cdots a_n'\quad(n\ge 0)
$$

- `S` 是串名。
- 单引号括起来的字符序列是串值；字符可以是字母、数字、标点或其他字符。
- 串中字符个数 `n` 称为串长。
- `n = 0` 时为空串，常记作 $\emptyset$。

注意区分：

| 概念 | 含义 |
|---|---|
| 空串 | 长度为 0，不含任何字符 |
| 空格串 | 由一个或多个空格字符组成，长度不为 0 |

考研语境里，串中字符位置通常从 1 开始计数，而不是从 0 开始。

## 子串、主串和位置

- 子串：串中任意个连续字符组成的子序列。
- 主串：包含该子串的串。
- 字符在主串中的位置：该字符第一次出现时在串中的序号。
- 子串在主串中的位置：子串第一个字符在主串中的位置。

例如 `T = 'iPhone 11 Pro Max?'`：

- `'iPhone'`、`'Pro M'` 是 `T` 的子串。
- `'1'` 第一次出现的位置是 8。
- `'11 Pro'` 在 `T` 中的位置是 8。

## 串和线性表的关系

串是一种特殊的[[linear-list-definition-and-operations|线性表]]：

- 逻辑结构仍然是一对一的线性关系。
- 数据对象被限定为字符集。
- 增删改查等操作通常以“子串”为操作对象，而不是单个元素。

这一点是串和一般线性表最核心的区别。线性表常按单个元素操作；串通常围绕子串、模式串、主串进行操作。

## 基本操作

设 `S = "iPhone 11 Pro Max?"`，`W = "Pro"`。

| 操作 | 含义 |
|---|---|
| `StrAssign(&T, chars)` | 将字符序列 `chars` 赋给串 `T` |
| `StrCopy(&T, S)` | 由串 `S` 复制得到串 `T` |
| `StrEmpty(S)` | 判断 `S` 是否为空串 |
| `StrLength(S)` | 返回 `S` 的字符个数 |
| `ClearString(&S)` | 将 `S` 清为空串 |
| `DestroyString(&S)` | 销毁串并回收存储空间 |
| `Concat(&T, S1, S2)` | 用 `T` 返回 `S1` 和 `S2` 连接后的新串 |
| `SubString(&Sub, S, pos, len)` | 取 `S` 中从第 `pos` 个字符起、长度为 `len` 的子串 |
| `Index(S, T)` | 返回模式串 `T` 在主串 `S` 中第一次出现的位置；不存在则返回 0 |
| `StrCompare(S, T)` | 按字符编码序比较两个串 |

例：

- `Concat(&T, S, W)` 得到 `"iPhone 11 Pro Max?Pro"`，实际实现时要考虑 `T` 的存储空间是否足够。
- `SubString(&T, S, 4, 6)` 得到 `"one 11"`。
- `Index(S, W)` 返回 11。

## 串比较

`StrCompare(S, T)` 从第一个字符开始逐个比较：

- 先出现更大字符的串更大。
- 若一个串是另一个串的前缀，则较长串更大。
- 只有两个串长度相等且每个字符都相同，才相等。

例如：

- `"abandon" < "aboard"`
- `"abstract" < "abstraction"`
- `"abstract" < "abstract "`
- `"academic" > "abuse"`

## 字符集与编码

字符必须经过编码才能存入计算机：

- 字符集：字符的集合，例如 ASCII、Unicode。
- 编码方案：字符到二进制数据的映射规则，例如 UTF-8、UTF-16。
- 同一个字符集可以有多种编码方案。

乱码的本质是：写入时按一套编码解释字符，读取时按另一套编码解释同一串二进制数据。

考研数据结构题通常默认每个字符占 1B，除非题目特别说明。
