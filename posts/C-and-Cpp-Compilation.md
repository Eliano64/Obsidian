---
title: C and Cpp Compilation
tags:
  - Compiler
categories:
date: 2026-06-24 00:00:00
katex: true
---


# gcc 与 g++ 

| 工具    | 常见用途      | 说明                                  |
| ----- | --------- | ----------------------------------- |
| `gcc` | 编译 C 程序   | 也能根据后缀调用对应前端，但链接 C++ 程序时不如 `g++` 省心 |
| `g++` | 编译 C++ 程序 | 会自动链接 C++ 标准库                       |

经验规则：

- C 文件：`gcc main.c -o main`
- C++ 文件：`g++ main.cpp -o main`

# 从源码到可执行文件的四个阶段

C/C++ 源码变成可执行程序，一般分为四步：预处理、编译、汇编、链接。

```text
.c/.cpp 源码
   ↓ 预处理 -E
.i/.ii 预处理结果
   ↓ 编译 -S
.s 汇编代码
   ↓ 汇编 -c
.o 目标文件
   ↓ 链接
可执行文件
```

## 1. 预处理 Preprocessing

```bash
gcc -E file.c -o file.i
g++ -E file.cpp -o file.ii
```

预处理会：

- 展开 `#include` 文件；
- 替换 `#define` 宏；
- 处理条件编译，如 `#ifdef` / `#ifndef` / `#if`。

输出结果是已经展开宏和头文件的文本文件。

## 2. 编译 Compilation

```bash
gcc -S file.i -o file.s
g++ -S file.ii -o file.s
```

编译阶段把预处理后的 C/C++ 代码翻译成汇编代码，输出 `.s` 文件。

## 3. 汇编 Assembling

```bash
gcc -c file.s -o file.o
g++ -c file.s -o file.o
```

汇编阶段把 `.s` 汇编代码转换成目标文件 `.o`。目标文件已经包含机器码，但通常还不能直接运行，因为外部函数、全局符号、库函数等引用还没完全解析。

常用的“一步生成目标文件”写法：

```bash
gcc -c file.c -o file.o
g++ -c file.cpp -o file.o
```

## 4. 链接 Linking

```bash
gcc file.o -o program
g++ file.o -o program
```

链接阶段会：

- 合并多个 `.o` 文件；
- 链接静态库或动态库；
- 解析函数调用、全局变量等符号地址；
- 生成最终可执行文件。

多个源文件的常见编译方式：

```bash
g++ -c main.cpp -o main.o
g++ -c util.cpp -o util.o
g++ main.o util.o -o app
```

# 常见组合命令

一步到位编译：

```bash
gcc file.c -o program
g++ file.cpp -o program
```

只预处理：

```bash
gcc -E file.c -o file.i
```

只生成汇编：

```bash
gcc -S file.c -o file.s
```

只编译不链接：

```bash
gcc -c file.c -o file.o
```

# 常用编译选项

| 选项                        | 描述                      |
| ------------------------- | ----------------------- |
| `-o <output_file>`        | 指定输出文件名；省略时通常输出 `a.out` |
| `-std=c++17` / `-std=c11` | 指定语言标准                  |
| `-O0`                     | 不优化，适合调试                |
| `-O2`                     | 常用优化级别，兼顾性能和稳定性         |
| `-O3`                     | 更激进优化，可能增加体积，也不一定总更快    |
| `-Os`                     | 优化体积                    |
| `-Wall`                   | 启用大部分常用警告               |
| `-Wextra`                 | 启用额外警告                  |
| `-Werror`                 | 把警告当作错误                 |
| `-g` / `-ggdb`            | 生成调试信息，便于 GDB 调试        |
| `-I<dir>`                 | 添加头文件搜索路径               |
| `-L<dir>`                 | 添加库文件搜索路径               |
| `-l<name>`                | 链接库，例如 `-lm` 链接数学库      |
| `-DNAME=value`            | 在编译命令中定义宏               |
| `-fstack-protector-all`   | 启用栈保护，尝试检测栈缓冲区溢出        |

开发调试阶段常用：

```bash
g++ -std=c++17 -O0 -g -Wall -Wextra -Werror -fsanitize=address,undefined main.cpp -o main
```

竞赛/发布阶段常用：

```bash
g++ -std=c++17 -O3 -Wall -Wextra main.cpp -o main
```

## Sanitizer

AddressSanitizer 和 UndefinedBehaviorSanitizer 很适合开发期排错。

```bash
g++ -g -O1 -fsanitize=address -fno-omit-frame-pointer main.cpp -o main
```

```bash
g++ -g -O1 -fsanitize=undefined main.cpp -o main
```

| 选项 | 检测内容 |
|---|---|
| `-fsanitize=address` | 越界访问、use-after-free 等内存访问错误 |
| `-fsanitize=undefined` | 未定义行为，如部分整数溢出、空指针解引用等 |

注意：Sanitizer 会增加编译时间和运行时开销，通常只在开发、测试、调试阶段使用。

# 常见错误定位

## undefined reference

示例：

```text
undefined reference to `foo()`
```

通常发生在链接阶段，表示声明存在，但链接器找不到实现。常见原因：

- 忘记把实现所在的 `.cpp` / `.o` 加入编译命令；
- 忘记链接库；
- 函数签名声明和定义不一致；
- C 与 C++ 混合链接时缺少 `extern "C"`。

## multiple definition

示例：

```text
multiple definition of `x`
```

通常表示同一个全局变量或函数被多个目标文件重复定义。常见原因：

- 在头文件中直接定义了非 `inline` 函数或全局变量；
- 头文件被多个 `.cpp` 包含后，每个 `.cpp` 都生成一份定义。

解决思路：

- 头文件里放声明：`extern int x;`
- 某一个 `.cpp` 里放定义：`int x = 0;`
- 函数模板、类模板通常需要放头文件；普通函数不要随便在头文件中定义，除非标记为 `inline`。

# 查看更详细帮助

```bash
man gcc
man g++
gcc --help
g++ --help
```
