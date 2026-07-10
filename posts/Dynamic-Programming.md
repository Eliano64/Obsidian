---
title: Dynamic Programming
tags:
  - DataStructureAndAlgorithm
  - DynamicProgramming
categories:
  - Data Structure & Algorithm
date: 2026-06-24 00:00:00
katex: true
---

# Dynamic Programming Notes

## 1. 动规五部曲

做动态规划题时，可以按下面五步检查：

1. **找 `dp[i]` 或 `dp[i][j]` 的含义。**
   - 先用一句话说清楚状态表示什么。
   - 状态没定义清楚时，不要急着写递推。
2. **初始化，并思考 `dp[0]`、`dp[1]` 等边缘情况。**
   - 注意题目数组长度的最小值。
   - 尤其要考虑边界越界的情况。
3. **找中间状态的递推公式。**
   - 当前状态从哪些更小或更早的状态转移过来？
   - 是取最大值、最小值、累加方案数，还是维护真假？
4. **根据递推公式和思路，确定遍历顺序。**
   - 0-1 背包容量倒序。
   - 完全背包容量正序。
   - 区间 DP 按区间长度从小到大。
   - 树形 DP 常用 DFS 回溯。
5. **检查：必要时打印 DP 数组。**
   - `dp` 可以是一维数组，也可以是二维数组 `dp[i][j]`。
   - 调试时打印小样例的 DP 表，往往能发现初始化或遍历顺序问题。

## 2. 0-1 背包

### 2.1 状态定义


```text
dp[i][j]：表示装第 i 件物品为止，在总容量不超过 j 的情况下，物品的最大总价值。
```

二维转移：

```text
dp[i][j] = max(dp[i-1][j], dp[i-1][j-capacity[i]] + value[i])
```

如果容量不够装第 `i` 件物品：

```text
dp[i][j] = dp[i-1][j]
```

[[0-1-knapsack-dp|空间优化]]

### 2.2 代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct goods
{
    int weight, value;
};

int main()
{
    vector<goods> tasks;
    int n;
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        int w, v;
        cin >> w >> v;
        tasks.push_back({w, v});
    }

    int capacity;
    cin >> capacity;

    // dp[j]：容量为 j 时的最大价值
    vector<int> dp(capacity + 1, 0);

    for (const auto& task : tasks)
    {
        for (int j = capacity; j >= task.weight; j--)
        {
            dp[j] = max(dp[j], dp[j - task.weight] + task.value);
        }
    }

    cout << dp[capacity];
}
```

### 2.3 为什么第二层循环要倒序？

这里的关键提醒：

> 注意循环嵌套的顺序，以及第二层的倒序。因为这是一个 0-1 背包问题，如果第二层为顺序的话，已经考虑过的 `dp[i]` 会在后面的 `dp[i+a]` 中被重复考虑。因此逆序。

换句话说，0-1 背包里每个物品最多用一次。倒序遍历容量，可以保证本轮转移使用的是“处理当前物品之前”的状态。

如果正序遍历，就会让当前物品在同一轮中被重复使用，变成完全背包。

### 2.4 两个相关提醒

还有两个常见相关点：

- 有一种 0-1 背包，`dp[i]` 表示数字之和不超过 `i` 的最大数。这相当于 `value = weight` 的情况。
- 多重背包可以先理解为：存在多个 `weight` 相同、`value` 相同的物品。更高效做法可以再用二进制拆分优化。

## 3. 0-1 背包变体：装满背包有几种方式

这一节求的不是最大价值，而是方案数。

### 3.1 状态定义

```text
dp[s]：使用已经遍历过的元素，凑出和 s 的方案数。
```

初始化：

```cpp
dp[0] = 1;
```

关键点：

> 什么都不选，也是一种方案。这里非常重要。

如果 `dp[0]` 不设为 1，后面所有方案数都无法从“空方案”推出来。

### 3.2 代码

```cpp
#include <iostream>
#include <vector>
using namespace std;

int solutions(vector<int>& fact, int target)
{
    vector<int> dp(target + 1, 0);
    dp[0] = 1;

    for (const auto& elem : fact)
    {
        for (int s = target; s >= elem; s--)
        {
            dp[s] += dp[s - elem];
            // 这个递推公式，可以联想爬梯子。
        }
    }

    // 0-1 背包经典：先物品，后容量，并且容量倒序遍历。
    return dp[target];
}

int main()
{
    int n;
    cin >> n;

    vector<int> fact;
    for (int i = 0; i < n; i++)
    {
        int elem;
        cin >> elem;
        fact.push_back(elem);
    }

    int target;
    cin >> target;
    cout << solutions(fact, target);
}
```

### 3.3 为什么仍然倒序？

因为这还是 0-1 背包：每个 `elem` 最多使用一次。

`dp[s] += dp[s - elem]` 的含义是：如果最后选择了 `elem`，那么之前必须已经凑出 `s - elem`。

## 4. 完全背包问题

完全背包和 0-1 背包的状态含义很像，但每个物品可以使用无限次。


```text
完全背包：dp[j] 表示在总容量不超过 j 的情况下，物品的最大总价值，初始 dp[0] = 0。
```

### 4.1 代码

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

struct goods
{
    int weight, value;
};

int main()
{
    vector<goods> tasks;
    int n;
    cin >> n;

    for (int i = 0; i < n; i++)
    {
        int w, v;
        cin >> w >> v;
        tasks.push_back({w, v});
    }

    int capacity;
    cin >> capacity;

    vector<int> dp(capacity + 1, 0);

    for (const auto& task : tasks)
    {
        for (int j = task.weight; j <= capacity; j++)
        {
            dp[j] = max(dp[j], dp[j - task.weight] + task.value);
        }
    }

    cout << dp[capacity];
}
```

### 4.2 和 0-1 背包的不同

注意：

> 注意跟 0-1 背包的不同之处。实际上完全背包的内外层关系可以随意反过来。这里为了与 0-1 一致，才也先物品后背包。真正的不同只有：背包容量是顺序遍历的。因为完全背包不需要考虑重复问题。

更准确地说：

- 如果求的是“最大价值”，完全背包常用先物品后容量，容量正序。
- 如果求方案数，还要区分组合数和排列数：
  - 先物品后容量：更像组合数；
  - 先容量后物品：更像排列数。

## 5. 最长公共子序列

子序列不需要在字符串或数组中连续。

### 5.1 状态定义

```text
dp[i][j]：str1 的前 i 个字符和 str2 的前 j 个字符的最长公共子序列长度。
```

### 5.2 代码

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    string str1, str2;
    cin >> str1 >> str2;

    vector<vector<int>> dp(str1.size() + 1, vector<int>(str2.size() + 1, 0));

    for (int i = 0; i < (int)str1.size(); i++)
    {
        for (int j = 0; j < (int)str2.size(); j++)
        {
            dp[i + 1][j + 1] = str1[i] == str2[j]
                ? dp[i][j] + 1
                : max(dp[i + 1][j], dp[i][j + 1]);
        }
    }

    cout << dp[str1.size()][str2.size()];
}
```

### 5.3 转移解释

如果 `str1[i] == str2[j]`，说明这两个字符可以共同作为一个公共子序列的结尾：

```text
dp[i+1][j+1] = dp[i][j] + 1
```

否则，只能从“去掉 str1 当前字符”或“去掉 str2 当前字符”两种情况中取最大值：

```text
dp[i+1][j+1] = max(dp[i+1][j], dp[i][j+1])
```

## 6. 最长公共子串

最长公共子串与最长公共子序列略有不同：

- 子序列不要求连续；
- 子串要求连续。

### 6.1 状态定义

关键点：

```text
dp[i][j]：长度为 i、j 的两字符串 s1、s2 中，且包括 s1[i-1]、s2[j-1] 的公共最长子串长度。
```

更简洁地说：

```text
dp[i][j] 表示以 s1[i-1] 和 s2[j-1] 结尾的最长公共后缀长度。
```

### 6.2 代码

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
using namespace std;

int main()
{
    string s1, s2;
    cin >> s1 >> s2;

    int memory = 0;
    vector<vector<int>> dp(s1.size() + 1, vector<int>(s2.size() + 1, 0));

    for (int i = 0; i < (int)s1.size(); i++)
    {
        for (int j = 0; j < (int)s2.size(); j++)
        {
            dp[i + 1][j + 1] = s1[i] == s2[j] ? dp[i][j] + 1 : 0;
            memory = max(memory, dp[i + 1][j + 1]);
        }
    }

    cout << memory;
}
```

### 6.3 为什么不相等时归零？

因为子串必须连续。如果当前位置字符不相等，那么“以这两个字符结尾”的公共子串不可能继续延伸，只能是 0。

## 7. 存在性 DP


```text
存在性 DP 的状态定义一般为：dp[i][j] 表示到第 i 个位置，是否存在元素 j，实际上是维护了一个值域较小的集合。
```

存在性 DP 通常用 `bool` 数组描述状态。它可以看作线性 DP 的一种变体。

### 7.1 更常见的表述

```text
dp[i][j]：考虑前 i 个元素，是否能达到状态 j。
```

例如，是否能从若干数中凑出和 `target`：

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canMakeTarget(vector<int>& nums, int target)
{
    vector<int> dp(target + 1, false);
    dp[0] = true;

    for (int x : nums)
    {
        for (int s = target; s >= x; s--)
        {
            dp[s] = dp[s] || dp[s - x];
        }
    }

    return dp[target];
}
```

这里倒序遍历，是因为每个数最多使用一次。

## 8. 区间 DP


> 区间动态规划解决的问题是通过小区间的计算逐步拓展到大区间，最后得出指定区间的最优解。

因此区间 DP 的状态转移方向是从小区间到大区间。

### 8.1 常见状态

```text
dp[l][r]：区间 [l, r] 上的最优解。
```

### 8.2 遍历顺序

最外层通常枚举区间长度，以确保转移方向正确：

```cpp
for (int len = 1; len <= n; len++)
{
    for (int l = 0; l + len - 1 < n; l++)
    {
        int r = l + len - 1;
        // 计算 dp[l][r]
    }
}
```

如果需要枚举断点：

```cpp
for (int len = 2; len <= n; len++)
{
    for (int l = 0; l + len - 1 < n; l++)
    {
        int r = l + len - 1;
        for (int k = l; k < r; k++)
        {
            dp[l][r] = min(dp[l][r], dp[l][k] + dp[k + 1][r] + cost(l, r));
        }
    }
}
```

### 8.3 复杂度提醒

区间 DP 需要枚举所有区间，因此时间复杂度一般较高。

常见复杂度：

```text
区间数 O(n^2) × 枚举断点 O(n) = O(n^3)
```

读题时要根据数据范围判断是否适合使用区间 DP。

## 9. 树形 DP


> 在树形动态规划中，我们通常通过深度优先搜索 DFS 遍历树，并使用数组或哈希表存储子问题的解，从而避免重复计算并提高算法效率。

树形 DP 常见方向：

- 自下而上树形 DP：通过子节点状态计算父节点状态。
- 自上而下树形 DP：通过父节点或祖先信息计算子节点状态。

### 9.1 自下而上树形 DP

自下而上树形 DP 的转移方向是从下往上。

基本思想：

- 状态定义：通常 `dp[u]` 表示以节点 `u` 为根的子树中，满足某种条件的最优解。
- 状态转移：对节点 `u` 的每个子节点 `v`，根据 `dp[v]` 更新 `dp[u]`。
- 边界条件：叶子节点没有子节点，可以直接计算。
- 遍历方式：通常使用 DFS。

### 9.2 第一类转移：子节点之间互不影响

第一类转移完全通过子节点更新父节点，且要求子节点之间互相不影响。

例如求子树大小：

```cpp
vector<vector<int>> g;
vector<int> sz;

void dfs(int u, int parent)
{
    sz[u] = 1;
    for (int v : g[u])
    {
        if (v == parent) continue;
        dfs(v, u);
        sz[u] += sz[v];
    }
}
```

例如求最大深度：

```cpp
vector<vector<int>> g;
vector<int> depthDown;

void dfs(int u, int parent)
{
    depthDown[u] = 0;
    for (int v : g[u])
    {
        if (v == parent) continue;
        dfs(v, u);
        depthDown[u] = max(depthDown[u], depthDown[v] + 1);
    }
}
```

### 9.3 第二类转移：子节点之间互相影响

在第二类转移中，子节点之间互相影响。

典型问题：要求以 `x` 为根的子树中，经过根的最长链长度。这时需要选择两个不同的子节点来计算结果。

如果计算完所有子节点信息后再暴力枚举两个子节点，复杂度可能较高。更好的方式是在遍历子节点时维护当前最大值和次大值，把转移控制在 `O(度数)`。

### 9.4 例题：树的直径 O(n)

树的直径是树中任意两个节点之间最长简单路径的长度。

对每个节点 `u`，维护两条从 `u` 向下走的最长链：

- `best1`：最长向下链；
- `best2`：次长向下链。

那么经过 `u` 的最长路径长度就是：

```text
best1 + best2
```

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> g;
int diameter = 0;

int dfs(int u, int parent)
{
    int best1 = 0;
    int best2 = 0;

    for (int v : g[u])
    {
        if (v == parent) continue;

        int d = dfs(v, u) + 1;

        if (d > best1)
        {
            best2 = best1;
            best1 = d;
        }
        else if (d > best2)
        {
            best2 = d;
        }
    }

    diameter = max(diameter, best1 + best2);
    return best1;
}

int main()
{
    int n;
    cin >> n;

    g.assign(n + 1, {});
    for (int i = 0; i < n - 1; i++)
    {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v);
        g[v].push_back(u);
    }

    dfs(1, 0);
    cout << diameter << '\n';
}
```

复杂度：每条边只被 DFS 访问常数次，所以是 `O(n)`。

## 10. 状压 DP

状压 DP（状态压缩动态规划）把一个集合状态压缩成二进制整数 `mask`，再用 `mask` 作为 DP 数组下标。它适合处理“从若干元素中选一些 / 排一个顺序 / 访问一批点”这类问题。

典型数据范围：

```text
n <= 20 左右
```

原因是状态总数为 `2^n`。当题目看起来需要枚举排列，暴力复杂度接近 `O(n!)`，但状态只与“已经选了哪些元素”有关时，就可以考虑状压 DP。

常见复杂度：

| 状态形式 | 适用场景 | 常见复杂度 |
|---|---|---|
| `dp[mask]` | 下一步是否合法只取决于已选集合和当前位置编号，不依赖上一个元素 | `O(n · 2^n)` |
| `dp[mask][last]` | 下一步是否合法依赖上一个选择的元素 `last` | `O(n^2 · 2^n)` |

### 10.1 `mask` 的基本含义

若有 `n` 个元素，可以用一个 `n` 位二进制数表示选择集合。

例如：

```text
mask = 5 = (101)_2
```

表示第 `0` 个和第 `2` 个元素已经被选中。

常用位运算：

| 操作 | 含义 |
|---|---|
| `mask & (1 << i)` | 判断第 `i` 个元素是否已经被选择 |
| `mask | (1 << i)` | 把第 `i` 个元素加入集合 |
| `mask ^ (1 << i)` | 翻转第 `i` 位，常用于删除或切换状态 |
| `__builtin_popcount(mask)` | 统计集合中有多少个元素 |

C++20 起也可以使用：

```cpp
#include <bit>

std::popcount((unsigned)mask);
```

竞赛环境里更常见的是：

```cpp
__builtin_popcount((unsigned)mask);
```

### 10.2 状态一：只记录已选集合 `dp[mask]`

当当前位置的合法性只与“当前要放第几个位置”和“已经选了哪些数”有关，而不需要知道上一个选的是谁时，可以只用一维状态：

```text
dp[mask]：已经选择集合 mask 时的方案数 / 最优值。
```

当前已经选了多少个元素，可以由 `popcount(mask)` 得到。因此下一个要填的位置通常是：

```text
pos = popcount(mask) + 1
```

这里 `pos` 是否从 1 开始，要看题目下标定义。

#### 10.2.1 例题：优美的排列

有从 `1` 到 `n` 的 `n` 个整数。构造排列 `perm`，下标从 `1` 开始。若对每个位置 `i`，满足下面至少一个条件，则称为优美排列：

- `perm[i]` 能被 `i` 整除；
- `i` 能被 `perm[i]` 整除。

约束：

```text
1 <= n <= 15
```

求优美排列数量。

状态：

```text
dp[mask]：已经把 mask 中的数字放入前 popcount(mask) 个位置时的方案数。
```

转移思路：

1. 当前已经放了 `cnt = popcount(mask)` 个数；
2. 下一个位置是 `pos = cnt + 1`；
3. 枚举还没选过的数字 `x = j + 1`；
4. 如果 `x` 和 `pos` 满足整除关系，就把它加入集合。

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

int countArrangement(int n)
{
    vector<int> dp(1 << n, 0);
    dp[0] = 1;

    for (int mask = 0; mask < (1 << n); mask++)
    {
        int pos = __builtin_popcount((unsigned)mask) + 1;

        for (int j = 0; j < n; j++)
        {
            if (mask & (1 << j)) continue;

            int x = j + 1;
            if (x % pos == 0 || pos % x == 0)
            {
                dp[mask | (1 << j)] += dp[mask];
            }
        }
    }

    return dp[(1 << n) - 1];
}
```

复杂度：

```text
状态数 2^n，每个状态枚举 n 个数：O(n · 2^n)
```

### 10.3 状态二：记录已选集合和最后一个元素 `dp[mask][last]`

如果下一步是否合法依赖“上一个选择的元素”，只知道集合 `mask` 不够，还要记录最后一个元素：

```text
dp[mask][last]：已经选择集合 mask，且最后一个选择的是 last 时的方案数 / 最优值。
```

例如：

```text
dp[5][2]
```

表示已经选了第 `0` 个和第 `2` 个元素，并且最后一个选择的是第 `2` 个。

因为：

```text
5 = (101)_2
```

#### 10.3.1 非法状态

`dp[mask][last]` 要求 `last` 必须在 `mask` 中。

例如：

```text
dp[6][0]
```

是不合法的，因为：

```text
6 = (110)_2
```

第 `0` 位不是 `1`，说明第 `0` 个元素没有被选中，因此不可能是“最后一个选择的元素”。

代码中通常这样跳过非法状态：

```cpp
if (!(mask & (1 << last))) continue;
```

### 10.4 例题：特别的排列

给定一个互不相同的正整数数组 `nums`。如果一个排列满足：对任意相邻两项 `a`、`b`，都有 `a % b == 0` 或 `b % a == 0`，则称为特别的排列。

约束：

```text
2 <= nums.length <= 14
1 <= nums[i] <= 1e9
```

求特别排列数量，对 `10^9 + 7` 取模。

这里相邻元素之间有约束，因此必须知道最后一个选择的是谁。

状态：

```text
dp[mask][i]：已经选择 mask 中的元素，且最后一个选择的是 nums[i] 时的方案数。
```

初始化：

```text
dp[1 << i][i] = 1
```

表示可以从任意一个元素开始。

转移：

```text
如果 j 还没被选择，并且 nums[i] 与 nums[j] 满足相邻条件：
dp[mask | (1 << j)][j] += dp[mask][i]
```

代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

int specialPerm(vector<int>& nums)
{
    const int MOD = 1e9 + 7;
    int n = nums.size();

    vector<vector<int>> dp(1 << n, vector<int>(n, 0));

    for (int i = 0; i < n; i++)
    {
        dp[1 << i][i] = 1;
    }

    for (int mask = 0; mask < (1 << n); mask++)
    {
        for (int last = 0; last < n; last++)
        {
            if (!(mask & (1 << last))) continue;
            if (dp[mask][last] == 0) continue;

            for (int nxt = 0; nxt < n; nxt++)
            {
                if (mask & (1 << nxt)) continue;

                if (nums[last] % nums[nxt] == 0 || nums[nxt] % nums[last] == 0)
                {
                    int nextMask = mask | (1 << nxt);
                    dp[nextMask][nxt] = (dp[nextMask][nxt] + dp[mask][last]) % MOD;
                }
            }
        }
    }

    int full = (1 << n) - 1;
    int ans = 0;
    for (int last = 0; last < n; last++)
    {
        ans = (ans + dp[full][last]) % MOD;
    }

    return ans;
}
```

复杂度：

```text
状态数 2^n × n，每个状态枚举下一个元素 n：O(n^2 · 2^n)
```

### 10.5 例题：旅行商变种，不回到起点

问题：

- 从原点 `(0, 0)` 出发；
- 有 `n` 个待访问点；
- 每个点都访问一次；
- 不要求回到起点；
- 路径长度按欧氏距离计算；
- 最终答案保留两位小数。

两点距离：

$$
d((x,y),(x',y')) = \sqrt{(x-x')^2 + (y-y')^2}
$$

这是典型的 `dp[mask][last]` 最短路型状压 DP。

状态：

```text
dp[mask][i]：已经访问 mask 中的所有点，并且最后访问的是第 i 个点时的最短距离。
```

初始化：

```text
dp[1 << i][i] = dist((0,0), point[i])
```

转移：

```cpp
int nextMask = mask | (1 << j);
dp[nextMask][j] = min(dp[nextMask][j], dp[mask][i] + dist(point[i], point[j]));
```

答案：

```text
min(dp[(1 << n) - 1][i])
```

因为不要求回到起点，所以最后不用再加回原点的距离。

完整代码：

```cpp
#include <bits/stdc++.h>
using namespace std;

double dist(const pair<int, int>& a, const pair<int, int>& b)
{
    double dx = a.first - b.first;
    double dy = a.second - b.second;
    return sqrt(dx * dx + dy * dy);
}

int main()
{
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    while (cin >> n)
    {
        vector<pair<int, int>> p(n);
        for (int i = 0; i < n; i++)
        {
            cin >> p[i].first >> p[i].second;
        }

        int total = 1 << n;
        const double INF = 1e18;
        vector<vector<double>> dp(total, vector<double>(n, INF));

        for (int i = 0; i < n; i++)
        {
            dp[1 << i][i] = dist({0, 0}, p[i]);
        }

        for (int mask = 0; mask < total; mask++)
        {
            for (int last = 0; last < n; last++)
            {
                if (!(mask & (1 << last))) continue;
                if (dp[mask][last] >= INF) continue;

                for (int nxt = 0; nxt < n; nxt++)
                {
                    if (mask & (1 << nxt)) continue;

                    int nextMask = mask | (1 << nxt);
                    dp[nextMask][nxt] = min(
                        dp[nextMask][nxt],
                        dp[mask][last] + dist(p[last], p[nxt])
                    );
                }
            }
        }

        int full = total - 1;
        double ans = INF;
        for (int last = 0; last < n; last++)
        {
            ans = min(ans, dp[full][last]);
        }

        cout << fixed << setprecision(2) << ans << '\n';
    }

    return 0;
}
```

复杂度：

```text
状态数：2^n × n
转移：每个状态枚举下一个点
总复杂度：O(n^2 · 2^n)
```

当 `n <= 15` 时通常可以接受。

### 10.6 常见QA

1. **`mask` 表示什么？**
   - 通常表示“已经选过 / 已经访问过”的集合。
2. **需不需要记录 `last`？**
   - 如果下一步和上一步无关，用 `dp[mask]`；
   - 如果下一步和上一个元素有关，用 `dp[mask][last]`。
3. **初始化是什么？**
   - 计数问题常见 `dp[0] = 1`；
   - 路径问题常见 `dp[1 << i][i] = 起点到 i 的代价`。
4. **非法状态有没有跳过？**
   - `dp[mask][last]` 中，必须保证 `last` 在 `mask` 里。
5. **答案从哪里收集？**
   - 计数排列通常看 `dp[(1 << n) - 1]` 或累加 `dp[full][last]`；
   - 最短路型问题通常对所有结尾 `last` 取最小值。

## 11. 期望 DP


```text
dp[i] 表示从 i 到终点的代价、价值的期望或概率。
```

期望 DP 常通过找 `dp[i]` 与相邻状态之间的关系来推导转移式。

### 11.1 原地不动模型

问题：从位置 `0` 出发，在位置 `i` 操作时：

- 有 `p_i` 的概率走到下一个位置 `i+1`；
- 有 `1-p_i` 的概率原地不动；
- 求走到 `n` 的期望操作次数。

设：

```text
dp[i]：从位置 i 到 n 所需的期望操作次数。
```

边界：

```text
dp[n] = 0
```

如果从 `i` 出发：

```text
dp[i] = 1 + p_i * dp[i+1] + (1-p_i) * dp[i]
```

把右边的 `dp[i]` 移到左边：

```text
p_i * dp[i] = 1 + p_i * dp[i+1]
```

所以：

```text
dp[i] = 1 / p_i + dp[i+1]
```

### 11.2 差分角度推导

也可以从 `dp[i] - dp[i-1]` 的角度分析：

```text
dp[i] - dp[i-1]
```

表示从 `i-1` 走到 `i` 的期望步数。

有：

$$
dp[i]-dp[i-1]=1+p_{i-1}\cdot 0+(1-p_{i-1})(dp[i]-dp[i-1])
$$

令：

```text
x = dp[i] - dp[i-1]
```

则：

```text
x = 1 + (1-p) x
p x = 1
x = 1 / p
```

这就是“先走一步，成功则到达，失败则回到原状态”的期望移项。

### 11.3 跳房子问题

例题：

> 小 e 从位置 1 出发，每次可以往右跳一格。但是每个房子摩擦系数不同，从房子 i 起跳时，有 `p_i` 的概率因为脚滑回到起点。求跳到终点的期望次数，结果对 `10^9+7` 取模。

这类题比“原地不动”更复杂，因为失败后不是留在当前位置，而是回到起点。

可设：

```text
E[i]：从位置 i 到 n 的期望次数。
```

对于 `i < n`：

```text
E[i] = 1 + (1-p_i) * E[i+1] + p_i * E[1]
```

由于所有状态都可能依赖 `E[1]`，可以把每个 `E[i]` 表示成：

```text
E[i] = a_i * E[1] + b_i
```

从后往前递推 `a_i` 和 `b_i`，最后在 `i=1` 处解方程：

```text
E[1] = a_1 * E[1] + b_1
E[1] = b_1 / (1 - a_1)
```

如果题目要求对 `10^9+7` 取模，除法要用乘法逆元。
