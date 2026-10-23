---
title: Cpp STL
tags:
  - STL
categories:
  - Cpp
date: 2026-06-24 00:00:00
katex: true
---

# 1. 环境与输入输出准备

## 1.1. 输入输出提速

语法：`ios::sync_with_stdio(false);`、`cin.tie(nullptr);`

`cin` / `cout` 默认与 C 标准输入输出保持同步，且 `cin` 每次读取前都会刷新 `cout`，数据量大时开销明显。关闭这两项可以显著提速：

```cpp
#include <iostream>
using namespace std;
int main() {
    ios::sync_with_stdio(false); // 关闭 iostream 与 stdio 的同步
    cin.tie(nullptr);            // 解除 cin 对 cout 的绑定
    // cout.tie(nullptr) 影响很小，可以省略
}
```

`ios::sync_with_stdio(false)` 之后不能再混用 `scanf` / `printf` 与 `cin` / `cout`，两者的缓冲区彼此独立，混用时读取顺序可能与预期不同。这两条设置都必须在任何输入输出之前执行。

## 1.2. 万能头文件

语法：`#include <bits/stdc++.h>`

```cpp
#include <bits/stdc++.h>
using namespace std;
```

`bits/stdc++.h` 是 GNU 扩展头文件，一次包含几乎所有常见标准库头文件。代价是它不属于 C++ 标准，非 GNU 环境可能不提供；并且每个编译单元都要解析大量头文件，编译更慢。需要移植或在意编译时间时按需包含：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <queue>
#include <map>
#include <functional>
```

## 1.3. 变量与类型

语法：`long long c = 1LL * a * b;`

数据范围可能超过 `int` 时用 `long long`，乘法最容易在类型提升上出错；内置类型的局部变量不初始化时其值不确定，声明时直接给初值比事后赋值更可靠。

```cpp
int a = 1000000000;
int b = 1000000000;
long long c = 1LL * a * b; // 正确：1LL 让整个表达式按 long long 计算
long long d = a * b;       // 错误：a * b 先按 int 计算并溢出，赋值已经太晚
```

# 2. 容器

## 2.1. vector

语法：`vector<T> v;`、`vector<T> v(n);`、`vector<T> v = {a, b};`

`vector` 是内存连续的顺序表，支持随机访问与尾部插入删除，适合元素个数会变化又需要频繁按下标访问的场景。

`vector` 扩容时会重新分配整块内存并搬移旧元素，扩容后所有指向原元素的指针、引用与迭代器全部失效。`push_back` 是均摊 `O(1)`，但触发扩容的那一次是 `O(n)`；已知规模时用 `reserve(n)` 或构造时指定大小可避免中途重分配。`resize(n)` 改变 `size()` 并构造元素，`reserve(n)` 只改变容量上限。

**成员函数参考**

| 函数 | 作用 | 复杂度 |
|---|---|---|
| `vector<T> v;` / `v(n)` / `v(n, x)` / `v = {a, b};` | 依次构造空向量、`n` 个值初始化元素、`n` 个值为 `x` 的元素、列表初始化 | `O(n)` |
| `push_back(x)` | 在尾部追加 `x` | 均摊 `O(1)` |
| `pop_back()` | 删除尾部元素 | `O(1)` |
| `operator[](i)` | 返回下标 `i` 的元素引用，不做越界检查 | `O(1)` |
| `front()` / `back()` | 返回首元素 / 尾元素的引用 | `O(1)` |
| `size()` / `empty()` | 元素个数 / 是否为空 | `O(1)` |
| `resize(n)` | 调整大小到 `n`，变大补值初始化元素，变小删尾部 | `O(n)` |
| `reserve(n)` | 预分配至少容纳 `n` 个元素的容量，不改变 `size()` | `O(n)` |
| `clear()` | 删除所有元素，`size()` 变为 0 | `O(n)` |
| `insert(it, x)` | 在迭代器 `it` 之前插入 `x` | `O(n)` |
| `erase(it)` | 删除迭代器 `it` 指向的元素 | `O(n)` |
| `begin()` / `end()` | 首元素迭代器 / 尾后迭代器 | `O(1)` |

**示例**

```cpp
#include <vector>
using namespace std;
int main() {
    vector<int> v = {3, 1, 4};
    v.reserve(100); // 预留容量，后续 push_back 不再重分配
    int* p = &v[0];
    v.push_back(5); // 未超过容量，p 仍有效
    v.pop_back();   // {3, 1, 4}
}
```

## 2.2. stack

语法：`stack<T> st;`

`stack` 是后进先出（LIFO）的容器适配器，默认以 `deque` 为底层容器，只允许在栈顶一端操作。

`pop()` 只删除不返回，取元素必须先 `top()` 再 `pop()`。两者都要求栈非空，对空栈调用是未定义行为。

**成员函数参考**

| 函数 | 作用 | 复杂度 |
|---|---|---|
| `push(x)` | 将 `x` 压入栈顶 | `O(1)` |
| `pop()` | 弹出栈顶元素，但不返回它 | `O(1)` |
| `top()` | 返回栈顶元素的引用，不弹出 | `O(1)` |
| `size()` / `empty()` | 元素个数 / 是否为空 | `O(1)` |

**示例**

```cpp
#include <stack>
#include <iostream>
using namespace std;
int main() {
    stack<int> st;
    st.push(1);
    st.push(2);
    while (!st.empty()) {        // 先判空，再取栈顶
        cout << st.top() << ' '; // 2 1
        st.pop();
    }
}
```

## 2.3. queue

语法：`queue<T> q;`

`queue` 是先进先出（FIFO）的容器适配器，默认以 `deque` 为底层容器，在队尾入队、队头出队。

`front()`、`back()`、`pop()` 都要求队列非空。

**成员函数参考**

| 函数 | 作用 | 复杂度 |
|---|---|---|
| `push(x)` | 将 `x` 从队尾入队 | `O(1)` |
| `pop()` | 弹出队头元素，但不返回它 | `O(1)` |
| `front()` | 返回队头元素的引用，不删除 | `O(1)` |
| `back()` | 返回队尾元素的引用，不删除 | `O(1)` |
| `size()` / `empty()` | 元素个数 / 是否为空 | `O(1)` |

**示例**（`queue` 没有 `clear()`，清空用空队列覆盖）

```cpp
#include <queue>
using namespace std;
int main() {
    queue<int> q;
    q.push(1);
    q = queue<int>(); // 用空队列覆盖原对象
}
```

## 2.4. deque

语法：`deque<T> dq;`

`deque` 是双端队列，内部由多段连续内存拼接，支持两端插入删除与随机访问，整体内存不连续。

两端插入删除会移动内部段指针，因此不保证指向元素的迭代器在两端操作后仍有效。

**成员函数参考**

| 函数 | 作用 | 复杂度 |
|---|---|---|
| `push_front(x)` | 从队头插入 `x` | `O(1)` |
| `push_back(x)` | 从队尾插入 `x` | `O(1)` |
| `pop_front()` | 弹出队头元素，但不返回它 | `O(1)` |
| `pop_back()` | 弹出队尾元素，但不返回它 | `O(1)` |
| `front()` / `back()` | 返回队头 / 队尾元素的引用 | `O(1)` |
| `operator[](i)` | 返回下标 `i` 的元素引用 | `O(1)` |
| `size()` / `empty()` | 元素个数 / 是否为空 | `O(1)` |
| `clear()` | 删除所有元素 | `O(n)` |

**示例**

```cpp
#include <deque>
#include <iostream>
using namespace std;
int main() {
    deque<int> dq;
    dq.push_back(2);
    dq.push_front(1);                               // {1, 2}
    dq.push_back(3);                                // {1, 2, 3}
    dq.pop_front();                                 // {2, 3}
    cout << dq.front() << ' ' << dq.back() << '\n'; // 2 3
}
```

## 2.5. priority_queue

语法：`priority_queue<T> pq;`、`priority_queue<T, vector<T>, Compare> pq;`

`priority_queue` 是优先队列，默认以 `vector` 为底层容器、用堆维护，默认是大顶堆，即最大元素在堆顶。它只保证 `top()` 是当前优先级最高者，其余元素不保证有序。

模板参数依次是元素类型、底层容器、比较器。自定义比较器返回 `bool`，返回真表示第一个参数的优先级比第二个低，于是第一个参数会下沉；因此 `[](const int& lhs, const int& rhs) { return lhs > rhs; }` 得到小顶堆。`priority_queue` 不提供迭代器，不能用 `begin()` / `end()` 或范围 for 遍历，也不能直接修改队列内的元素；`top()` 与 `pop()` 都要求非空。

**成员函数参考**

| 函数 | 作用 | 复杂度 |
|---|---|---|
| `push(x)` | 插入 `x` 并调整堆 | `O(log n)` |
| `pop()` | 删除堆顶元素，但不返回它 | `O(log n)` |
| `top()` | 返回堆顶元素的引用，不删除 | `O(1)` |
| `size()` / `empty()` | 元素个数 / 是否为空 | `O(1)` |

**示例**（默认比较器 `less<T>` 是大顶堆，传入 `greater<T>` 得到小顶堆）

```cpp
#include <queue>
#include <vector>
#include <functional>
#include <iostream>
using namespace std;
int main() {
    priority_queue<int> mx;                            // 大顶堆
    priority_queue<int, vector<int>, greater<int>> mn; // 小顶堆
    mx.push(1); mx.push(5); mx.push(3);
    mn.push(1); mn.push(5); mn.push(3);
    cout << mx.top() << ' ' << mn.top() << '\n'; // 5 1
}
```

## 2.6. map 与 unordered_map

语法：`map<K, V> mp;`、`unordered_map<K, V> um;`

`map` 是基于红黑树的关联容器，键唯一且按键有序，查找、插入、删除都是 `O(log n)`，需要有序遍历键时使用。`unordered_map` 基于哈希表，平均 `O(1)` 完成查找插入，最坏退化到 `O(n)`，迭代顺序不保证。

`unordered_map` 的平均 `O(1)` 依赖哈希分布均匀，大量键落进同一桶会退化成 `O(n)`；迭代顺序由桶结构与哈希值决定，不要依赖。用自定义结构体作键时，`map` 要求 `operator<` 或比较器，`unordered_map` 要求自定义哈希函数。

**成员函数参考**

| 函数 | 作用 | map 复杂度 | unordered_map 复杂度 |
|---|---|---|---|
| `insert({key, val})` | 插入键值对，键已存在时不覆盖 | `O(log n)` | 平均 `O(1)`，最坏 `O(n)` |
| `erase(key)` | 删除指定键的键值对 | `O(log n)` | 平均 `O(1)`，最坏 `O(n)` |
| `find(key)` | 返回指向该键的迭代器，不存在返回 `end()` | `O(log n)` | 平均 `O(1)`，最坏 `O(n)` |
| `at(key)` | 返回该键对应值的引用，键不存在时抛 `std::out_of_range` | `O(log n)` | 平均 `O(1)` |
| `operator[](key)` | 键不存在时插入值初始化元素并返回其引用 | `O(log n)` | 平均 `O(1)` |
| `count(key)` | 键的个数，键唯一时只可能是 0 或 1 | `O(log n)` | 平均 `O(1)` |
| `size()` / `empty()` | 键值对个数 / 是否为空 | `O(1)` | `O(1)` |
| `clear()` | 删除所有键值对 | `O(n)` | `O(n)` |
| `begin()` / `end()` | 首元素迭代器 / 尾后迭代器 | `O(1)` | `O(1)` |

**示例**（只查询必须用 `find()` 或 `at()`，`operator[]` 会在键不存在时插入元素）

```cpp
#include <map>
#include <iostream>
using namespace std;
int main() {
    map<int, int> mp;
    mp[1] = 10;                // 插入 {1, 10}
    if (mp.find(2) == mp.end()) cout << "2 不存在\n";
    cout << mp.size() << '\n'; // 1
    cout << mp[2] << '\n';     // 0，operator[] 已插入 {2, 0}
    cout << mp.size() << '\n'; // 2
}
```

## 2.7. set、multiset 与 unordered_set

语法：`set<T> s;`、`multiset<T> ms;`、`unordered_set<T> us;`

`set` 是基于红黑树的有序集合，元素唯一，默认升序；`multiset` 允许重复元素；`unordered_set` 基于哈希表，元素唯一但不保持顺序。`set` 的关键字就是元素本身，容器内元素只读：改动会破坏树的有序不变量，所以「修改」一个元素必须先用 `find()` 定位并 `erase()`，再 `insert()` 新值。`erase(it)` 要求 `it` 有效且非 `end()`。

自定义比较器是返回 `bool` 的函数对象，定义的是严格弱序而非哈希，例如 `struct cmp { bool operator()(const int& u, const int& v) const { return u > v; } };` 配上 `set<int, cmp>` 即降序排列。需要哈希去重时用 `unordered_set` 并提供哈希函数；其 `insert`、`erase`、`find`、`count` 平均 `O(1)`，最坏 `O(n)`，遍历顺序不保证。

**成员函数参考**

| 函数 | 作用 | set | multiset |
|---|---|---|---|
| `insert(x)` | 插入 `x` | `O(log n)`，已存在时无效果 | `O(log n)`，允许重复 |
| `erase(val)` | 按值删除 | 删除该元素，`O(log n)` | 删除所有等于 `val` 的元素，`O(k + log n)`，`k` 为个数 |
| `erase(it)` | 删除迭代器指向的元素 | 均摊 `O(1)` | 均摊 `O(1)` |
| `find(x)` | 返回指向 `x` 的迭代器，不存在返回 `end()` | `O(log n)` | `O(log n)` |
| `count(x)` | 返回 `x` 的个数 | 0 或 1 | 出现次数 |
| `size()` / `empty()` | 元素个数 / 是否为空 | `O(1)` | `O(1)` |
| `clear()` | 删除所有元素 | `O(n)` | `O(n)` |
| `begin()` / `end()` | 首元素迭代器 / 尾后迭代器，按键序 | `O(1)` | `O(1)` |

**示例**

```cpp
#include <set>
#include <iostream>
using namespace std;
int main() {
    set<int> s;
    s.insert(3); s.insert(1); s.insert(1); // 1 重复，无效果
    cout << s.size() << '\n';              // 2
    auto it = s.find(3);
    if (it != s.end()) s.erase(it);        // 先删除
    s.insert(2);                           // 再插入新元素
    multiset<int> ms;                      // 允许重复
    ms.insert(2); ms.insert(2);
    cout << ms.count(2) << '\n';           // 2
    ms.erase(2);                           // 按值删除全部 2
}
```

## 2.8. bitset

语法：`bitset<N> b;`

`bitset<N>` 是位宽固定为 `N` 的位序列，适合存储和批量运算二进制状态，初始化后各位默认都是 0。位宽 `N` 必须是编译期常量，运行时确定的长度不能用它。下标 0 是最低位，与整数的二进制写法一致。

标注 `O(N)` 的操作实际按机器字打包处理，常数很小，通常可当作 `O(N / w)`，其中 `w` 是机器字长。

**成员函数与运算参考**

| 成员 / 运算 | 作用 | 复杂度 |
|---|---|---|
| `bitset<N> b;` / `b(x)` / `b(string("101"))` | 依次构造全 0 序列、用整数 `x` 的二进制构造、用字符串构造（右侧为低位） | `O(N)` |
| `operator[](i)` | 返回第 `i` 位的引用 | `O(1)` |
| `set()` / `set(i)` | 全部置 1 / 第 `i` 位置 1 | `O(N)` / `O(1)` |
| `reset()` / `reset(i)` | 全部置 0 / 第 `i` 位置 0 | `O(N)` / `O(1)` |
| `flip()` / `flip(i)` | 全部取反 / 第 `i` 位取反 | `O(N)` / `O(1)` |
| `count()` | 值为 1 的位数 | `O(N)` |
| `size()` | 位宽 `N` | `O(1)` |
| `any()` / `none()` / `all()` | 是否存在 1 / 是否全 0 / 是否全 1 | `O(N)` |
| `to_string()` | 转为由 `0`、`1` 组成的字符串 | `O(N)` |
| `&`、`\|`、`^`、`~`、`<<`、`>>` | 逐位运算与移位，两侧位宽须相同 | `O(N)` |

**示例**

```cpp
#include <bitset>
#include <iostream>
using namespace std;
int main() {
    bitset<8> b(5);                // 00000101
    b.set(7);                      // 10000101
    b.flip(0);                     // 10000100
    cout << b.count() << '\n';     // 2
    cout << b.to_string() << '\n'; // 10000100
}
```

## 2.9. string

语法：`string s = "text";`

`string` 是可动态增长的字符序列，内部连续存储，提供连接、查找、替换、插入、删除、取子串等操作，用法类似动态字符数组。

`find` 系列找不到目标时返回 `string::npos`，它是一个 `size_t` 特殊值（通常为无符号最大值），判断必须写成 `s.find(t) == string::npos`，不能拿它当普通下标比较大小。频繁拼接或修改会反复扩容搬移数据，已知长度时用 `reserve(n)` 减少重分配。

**成员函数参考**

| 成员 / 运算 | 作用 | 复杂度 |
|---|---|---|
| `length()` / `size()` | 返回字符个数，两者等价 | `O(1)` |
| `empty()` | 是否为空串 | `O(1)` |
| `substr(pos, len)` | 返回从 `pos` 起、长度为 `len` 的子串；省略 `len` 到末尾 | `O(len)` |
| `erase(pos, len)` | 删除从 `pos` 起、长度为 `len` 的字符 | `O(len)` |
| `insert(pos, str)` | 在 `pos` 位置插入字符串 | `O(n)` |
| `replace(pos, len, str)` | 用 `str` 替换从 `pos` 起长度为 `len` 的区间 | `O(n)` |
| `push_back(c)` / `pop_back()` | 尾部追加字符 / 删除尾字符 | 均摊 `O(1)` / `O(1)` |
| `front()` / `back()` | 返回首字符 / 尾字符的引用 | `O(1)` |
| `operator+` | 拼接字符或字符串 | `O(len)` |
| `operator==`、`<`、`<=`、`>`、`>=` | 按字典序比较 | `O(n)` |
| `find(str)` | 返回子串或字符首次出现的位置，找不到返回 `npos` | `O(nm)` |
| `c_str()` | 返回等价的 C 风格字符指针 | `O(1)` |
| `reserve(n)` | 预留至少容纳 `n` 个字符的容量 | `O(n)` |

**示例**

```cpp
#include <string>
#include <iostream>
using namespace std;
int main() {
    string s = "Hello World";
    cout << s.substr(0, 5) << '\n';                 // Hello
    size_t p = s.find("World");
    if (p != string::npos) s.replace(p, 5, "Cpp");  // Hello Cpp
    s.push_back('!');
    cout << s << ' ' << s.size() << '\n';           // Hello Cpp! 10
}
```

## 2.10. pair

语法：`pair<A, B> p{a, b};`

`pair<A, B>` 把两个值绑成一个整体，两个公有成员是 `first` 与 `second`，类型可以不同，适合返回两个值或作为容器元素。

| 用法 | 示例 | 说明 |
|---|---|---|
| 声明 | `pair<int, string> p;` | 默认构造，成员值初始化 |
| 构造 | `pair<int, int> p = make_pair(1, 2);` | 也可写 `{1, 2}` |
| 访问 | `p.first`、`p.second` | 直接读写两个成员 |
| 比较 | `p1 < p2` | 先比较 `first`，相等再比较 `second` |
| 返回多个值 | `pair<int, int> f(...)` | 调用方取 `.first` 与 `.second` |
| 作为元素 | `vector<pair<int, int>> v;` | 与容器组合使用 |

**示例**

```cpp
#include <utility>
#include <vector>
#include <iostream>
using namespace std;
pair<int, int> minmaxOf(int x, int y) {
    if (x > y) swap(x, y);
    return {x, y};                             // 返回两个值
}
int main() {
    cout << (make_pair(1, 2) < make_pair(1, 3)) << '\n'; // 1
    auto q = minmaxOf(7, 3);
    cout << q.first << ' ' << q.second << '\n';          // 3 7
}
```

# 3. 算法

## 3.1. sort

语法：`sort(first, last);`、`sort(first, last, cmp);`

`sort` 定义在 `<algorithm>`，对随机访问迭代器区间 `[first, last)` 排序，默认升序，时间复杂度 `O(n log n)`。

`sort` 不是稳定排序，相等元素的原有相对顺序可能改变，需要保留时用 `stable_sort`。比较器必须满足严格弱序：对任意元素 `a`，`cmp(a, a)` 必须为假，因此写成 `a <= b` 会在相等元素上返回真，破坏序关系并导致未定义行为，比较应写 `a < b`。`sort` 需要随机访问迭代器，适用于数组、`vector`、`deque`，不能用于 `list` 或 `forward_list`。

```cpp
#include <algorithm>
#include <vector>
using namespace std;
int main() {
    vector<int> v = {3, 1, 4, 1, 5};
    sort(v.begin(), v.end());                                     // 升序
    sort(v.begin(), v.end(), [](int a, int b) { return a > b; }); // 降序
}
```

## 3.2. lower_bound 与 upper_bound

语法：`lower_bound(first, last, v);`、`upper_bound(first, last, v);`

两者定义在 `<algorithm>`，在已按同一比较规则排好序的区间 `[first, last)` 上做二分查找，返回迭代器，复杂度 `O(log n)`。输入未排序时结果不保证正确。

有序关联容器 `map`、`set` 自带同名的成员函数 `lower_bound`、`upper_bound`，它们直接利用树结构查找，复杂度 `O(log n)` 且不要求随机访问迭代器；对这类容器应优先调用成员版本，而不是把整个容器交给自由函数版本。

| 项目 | `lower_bound(first, last, val)` | `upper_bound(first, last, val)` |
|---|---|---|
| 返回位置 | 第一个不小于 `val` 的元素，即第一个 `>= val` | 第一个大于 `val` 的元素，即第一个 `> val` |
| 存在多个等于 `val` 的元素 | 指向其中第一个 | 跳过全部 `val`，指向其后更大的元素 |
| 返回 `last` 的条件 | 所有元素都 `< val` | 所有元素都 `<= val` |
| 复杂度 | `O(log n)` | `O(log n)` |

**示例**（两个迭代器相减即 `val` 的出现次数）

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;
int main() {
    vector<int> v = {1, 3, 3, 5, 7};
    auto l = lower_bound(v.begin(), v.end(), 3); // 下标 1
    auto r = upper_bound(v.begin(), v.end(), 3); // 下标 3
    cout << (r - l) << '\n';                     // 2，3 的出现次数
}
```

## 3.3. next_permutation 与 prev_permutation

语法：`next_permutation(first, last);`、`prev_permutation(first, last);`

两个函数定义在 `<algorithm>`，按字典序把区间改写成下一个 / 上一个排列，返回是否成功，复杂度 `O(n)`。`next_permutation` 在当前排列已是最大字典序时返回 `false` 并把区间改写回最小字典序（即升序排列）。要枚举全部 `n!` 个排列必须先排序，再用 `do { ... } while (...)`：循环体先处理最小排列，返回 `false` 时序列已复位，循环恰好结束。

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
using namespace std;
int main() {
    vector<int> a = {2, 1, 3};
    sort(a.begin(), a.end()); // 必须先升序，才能覆盖全部排列
    do {
        for (int x : a) cout << x;
        cout << '\n';         // 123 132 213 231 312 321
    } while (next_permutation(a.begin(), a.end()));
}
```

`prev_permutation` 对称：在原序列已是最小字典序时返回 `false` 并改写为最大字典序；从最大排列开始枚举全部排列时应先降序排列。

## 3.4. reverse

语法：`reverse(first, last);`

`reverse` 定义在 `<algorithm>`，把区间 `[first, last)` 内的元素顺序颠倒，需要双向迭代器，复杂度 `O(n)`。

```cpp
#include <algorithm>
#include <vector>
using namespace std;
int main() {
    vector<int> v = {1, 2, 3, 4, 5};
    reverse(v.begin(), v.end()); // {5, 4, 3, 2, 1}
}
```

数组、`vector`、`string`、`deque`、`list` 都满足双向迭代器要求，`forward_list` 因为只能单向前进而不适用。

## 3.5. swap

语法：`swap(a, b);`

`swap(a, b)` 交换两个同类型对象的值，不改变对象在内存中的位置。

```cpp
#include <utility>
using namespace std;
int main() {
    int a = 5, b = 10;
    swap(a, b); // a = 10，b = 5
}
```

对内置类型，`swap` 通过三步移动完成，复杂度 `O(1)`。对 `vector`、`string` 与关联容器，特化版本只交换内部指针与元数据，因此 `O(1)` 且不逐个搬移元素，指向被交换容器元素的迭代器与引用仍然有效，只是现在属于另一个对象。对自定义类型，复杂度取决于其移动或交换的实现。

# 4. lambda 与 std::function

## 4.1. 捕获列表

语法：`[capture](parameters) -> return_type { body }`

返回类型可以省略，由编译器从 `return` 语句推导。

按值捕获在 lambda 构造时拷贝变量，之后外部变量的改动不影响副本，副本的生命周期与 lambda 对象绑定。按引用捕获不拷贝，但 lambda 内保存的是外部变量的引用，外部变量先销毁再调用 lambda 会得到悬空引用，交给异步任务或长期存储的 lambda 不能捕获局部变量的引用。捕获会扩大 lambda 对象的体积并产生拷贝成本，只捕获真正用到的变量可避免多余开销。

| 写法 | 含义 |
|---|---|
| `[]` | 不捕获任何外部变量 |
| `[=]` | 按值捕获用到的外部变量 |
| `[&]` | 按引用捕获用到的外部变量 |
| `[x]` | 只按值捕获 `x` |
| `[&x]` | 只按引用捕获 `x` |
| `[=, &x]` | 默认按值，`x` 按引用 |
| `[&, x]` | 默认按引用，`x` 按值 |

**示例**

```cpp
#include <vector>
using namespace std;
int main() {
    int threshold = 2;
    vector<int> v = {1, 3, 5};
    // 按引用捕获，可在 lambda 内修改外部变量
    auto countAbove = [&threshold](const vector<int>& w) {
        ++threshold;
        int c = 0;
        for (int x : w) if (x > threshold) ++c;
        return c;
    };
}
```

## 4.2. lambda 的类型

语法：`auto f = [](int x) { return x * x; };`

每个 lambda 表达式都产生一个唯一的匿名闭包类，没有可直接书写的类型名，只能用 `auto` 接收：

```cpp
#include <iostream>
using namespace std;
int main() {
    auto square = [](int x) { return x * x; }; // auto 推导出唯一的闭包类型
    cout << square(5) << '\n';                 // 25
}
```

闭包类型不可默认构造，同一个 `auto` 变量只能保存同一处 lambda。需要把不同可调用对象存进同一变量、放进容器或作为参数传递时，用 `std::function` 做类型擦除。

## 4.3. 递归 lambda

语法：`function<void(int)> dfs = [&](int u) { ... };`、`auto dfs = [&](auto&& self, int u) { ... };`

lambda 在定义完成之前没有名字，函数体内无法直接调用自身，有两种写法。第一种用 `std::function` 先声明再赋值，`<functional>` 提供该类型：

```cpp
#include <functional>
#include <vector>
using namespace std;
int main() {
    vector<vector<int>> g(10);
    vector<int> vis(10, 0);
    function<void(int)> dfs = [&](int u) { // 通过已声明的 dfs 递归
        vis[u] = 1;
        for (int v : g[u]) if (!vis[v]) dfs(v);
    };
    dfs(1);
}
```

第二种把自身作为第一个参数传入，用 `auto&& self` 接收，避免 `std::function` 的类型擦除开销：

```cpp
#include <vector>
using namespace std;
int main() {
    vector<vector<int>> g(10);
    vector<int> vis(10, 0);
    auto dfs = [&](auto&& self, int u) -> void {
        vis[u] = 1;
        for (int v : g[u]) if (!vis[v]) self(self, v); // 把自身继续传下去
    };
    dfs(dfs, 1); // 入口必须显式带上 self
}
```

`std::function` 写法递归调用自然，但每次调用都经过一次间接跳转；`auto self` 写法由编译器按模板实例化出具体类型，可以直接内联，数据规模大、递归深时差距可见。
