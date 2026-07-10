---
title: Cpp Competitive Programming Idioms
tags:
  - Lambda
categories:
  - Cpp
date: 2026-06-24 00:00:00
katex: true
---

# Cpp Competitive Programming Idioms

## 1. C++ IO 提速


常用写法：

```cpp
ios::sync_with_stdio(0);
cin.tie(0);
cout.tie(0);
```


含义：

- `ios::sync_with_stdio(0)`：关闭 C++ iostream 与 C stdio 的同步，提高 `cin/cout` 速度。
- `cin.tie(0)`：解除 `cin` 和 `cout` 的绑定，避免每次输入前自动刷新输出。
- `cout.tie(0)`：通常影响不大，可写可不写。

注意：关闭同步后，不建议混用 `cin/cout` 与 `scanf/printf`。

## 2. 变量定义习惯


常见写法：

> 常用 `long long` 类型。

当数据范围可能超过 `int` 时，应使用 `long long`。尤其是乘法时，注意先提升类型：

```cpp
int a = 1e9;
int b = 1e9;
long long c = 1LL * a * b;
```

如果写成：

```cpp
long long c = a * b;
```

乘法可能先以 `int` 计算，溢出后再赋值给 `long long`，已经晚了。

常见命名习惯：

| 名字 | 常见含义 |
|---|---|
| `n, m` | 规模、点数边数、行列数 |
| `i, j, k` | 循环变量 |
| `u, v, w` | 图中的边：起点、终点、权重 |
| `ans` | 答案 |
| `cnt` | 计数 |
| `idx` | 下标或编号 |
| `l, r` | 左右边界 |

## 3. 万能头文件


常见写法：

```cpp
#include <bits/stdc++.h>
using namespace std;
```

`bits/stdc++.h` 是竞赛中常用的 GNU 扩展头文件，几乎包含常见标准库头文件。

优点：写起来快，竞赛环境通常支持。  
缺点：不是 C++ 标准，某些编译器或工程环境不支持。

工程代码建议按需包含：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>
#include <map>
#include <functional>
```

## 4. lambda 函数


`sort()` 中常见的 lambda 用法：

```cpp
sort(v.begin(), v.end(), [](int a, int b) {
    return a > b; // 降序排序
});
```

lambda 基本形式：

```cpp
[capture](parameters) -> return_type {
    body
}
```

返回类型通常可以省略：

```cpp
auto square = [](int x) {
    return x * x;
};
```

### 4.1 捕获列表

| 写法 | 含义 |
|---|---|
| `[]` | 不捕获外部变量 |
| `[=]` | 按值捕获用到的外部变量 |
| `[&]` | 按引用捕获用到的外部变量 |
| `[x]` | 只按值捕获 `x` |
| `[&x]` | 只按引用捕获 `x` |
| `[=, &x]` | 默认按值，`x` 按引用 |
| `[&, x]` | 默认按引用，`x` 按值 |

## 5. std::function

`std::function` 定义在 `<functional>`，可以保存“可调用对象”：普通函数、lambda、函数对象等。

### 5.1 基本用法

```cpp
#include <functional>

function<int(int, int)> add = [](int a, int b) {
    return a + b;
};

cout << add(1, 2); // 3
```

`function<int(int,int)>` 的含义是：保存一个可调用对象，它接收两个 `int` 参数，返回 `int`。

### 5.2 递归 lambda：std::function 写法

现在 C++ 中写 DFS 时，常用 `std::function` 保存递归 lambda：

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    cin >> n;

    vector<vector<int>> g(n + 1);
    vector<int> vis(n + 1, 0);

    function<void(int)> dfs = [&](int u) {
        vis[u] = 1;
        for (int v : g[u]) {
            if (!vis[v]) dfs(v);
        }
    };

    dfs(1);
}
```

为什么这里需要 `std::function`？因为 lambda 在定义时还没有名字，直接在 lambda 体内调用自己不方便。`std::function` 先声明一个可调用对象变量 `dfs`，lambda 捕获引用后就可以递归调用 `dfs(v)`。

### 5.3 递归 lambda：auto self 写法

如果想避免 `std::function` 的类型擦除开销，也可以使用 “self 参数” 写法：

```cpp
auto dfs = [&](auto&& self, int u) -> void {
    vis[u] = 1;
    for (int v : g[u]) {
        if (!vis[v]) self(self, v);
    }
};

dfs(dfs, 1);
```

比较：

| 写法 | 优点 | 缺点 |
|---|---|---|
| `std::function` | 写法直观，递归调用自然 | 有类型擦除开销，需包含 `<functional>` |
| `auto self` | 性能更好，无 `std::function` 开销 | 写法稍绕 |

竞赛中两种都常见：短代码、图搜索可以用 `std::function`；极限性能场景可用 `auto self`。
