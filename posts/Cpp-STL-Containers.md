---
title: Cpp STL Containers
tags:
  - DataStructureAndAlgorithm
  - STL
  - Container
categories:
  - Cpp
date: 2026-06-24 00:00:00
katex: true
---

# Cpp STL Containers Notes

## 1. vector


`vector` 是一个顺序表，内存空间连续，支持自动扩容和自动内存管理。

### 1.1 初始化

| 写法 | 含义 | 示例 |
|---|---|---|
| `vector<T> v;` | 声明一个空向量，元素类型为 `T` | `vector<double> v;` |
| `vector<int> v;` | 声明一个存储 `int` 的空向量 | `[]` |
| `vector<int> v(n);` | 声明大小为 `n` 的向量，默认初值为 0 | `n=5` 时 `[0,0,0,0,0]` |
| `vector<int> v(n, x);` | 声明大小为 `n`、初值为 `x` 的向量 | `n=5,x=3` 时 `[3,3,3,3,3]` |
| `vector<int> v = {1,2,3};` | 列表初始化 | `[1,2,3]` |

同一个向量中的元素类型必须一致。如果已确定所需数组大小，建议在声明时设定大小或使用 `reserve()`，避免频繁动态扩容。

### 1.2 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `push_back(x)` | 将元素 `x` 插入向量末尾 | `O(1)` 均摊 |
| `pop_back()` | 删除尾部元素 | `O(1)` |
| `operator[idx]` | 中括号操作符，返回下标 `idx` 所指元素 | `O(1)` |
| `front()` | 返回向量中第一个元素，即下标为 0 的元素 | `O(1)` |
| `back()` | 返回向量中的最后一个元素 | `O(1)` |
| `size()` | 返回向量中的元素个数 | `O(1)` |
| `empty()` | 判断向量是否为空 | `O(1)` |
| `resize(n)` | 将向量大小调整为 `n`，多删少补 | `O(n)` |
| `clear()` | 将向量清空 | `O(n)` |
| `erase(it)` | 删除迭代器 `it` 所指元素 | `O(n)` |
| `insert(it, val)` | 在迭代器 `it` 前插入 `val` | `O(n)` |
| `begin()` | 返回指向向量中第一个元素的迭代器 | `O(1)` |
| `end()` | 返回指向向量中最后一个元素的下一个位置的迭代器 | `O(1)` |

## 2. stack


`stack` 是遵循 FILO / LIFO（先进后出、后进先出）原则的数据结构。

### 2.1 初始化

```cpp
stack<T> stk;
stack<int> stk_int;
```

`T` 可以是 `int`、`double`、`char`，也可以是自定义结构体。同一个栈内元素类型必须一致。

### 2.2 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `push(x)` | 将元素 `x` 推入栈顶 | `O(1)` |
| `pop()` | 弹出栈顶元素，但不会返回该元素 | `O(1)` |
| `top()` | 返回栈顶元素，但不会弹出该元素 | `O(1)` |
| `size()` | 返回栈中的元素个数 | `O(1)` |
| `empty()` | 判断栈是否为空 | `O(1)` |

调用 `top()` 或 `pop()` 前应确保栈非空。

## 3. queue


`queue` 遵循先进先出（FIFO）原则，元素按照被添加的顺序移除。

### 3.1 初始化

```cpp
queue<T> q;
queue<int> qint;
```

### 3.2 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `push(x)` | 将元素 `x` 从队尾推入 | `O(1)` |
| `pop()` | 将队头元素弹出，但不返回该元素 | `O(1)` |
| `size()` | 返回队列大小，即队列中的元素个数 | `O(1)` |
| `empty()` | 判断队列是否为空，若为空返回 `true`，否则返回 `false` | `O(1)` |
| `front()` | 返回队头元素，但不删除该元素 | `O(1)` |
| `back()` | 返回队尾元素 | `O(1)` |

如果想清空队列，可以重新赋值为空队列：

```cpp
q = queue<int>();
```

## 4. deque


`deque` 是双端队列（double-ended queue），允许在队列两端添加和移除元素。把普通队列换成双端队列时，很多代码只需要把 `queue` 换成 `deque`。

### 4.1 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `push_front(x)` | 将元素 `x` 从队头推入 | `O(1)` |
| `push_back(x)` | 将元素 `x` 从队尾推入 | `O(1)` |
| `pop_front()` | 将队头元素弹出（删除），但不返回该元素 | `O(1)` |
| `pop_back()` | 将队尾元素弹出（删除），但不返回该元素 | `O(1)` |
| `size()` | 返回双端队列大小，即元素个数 | `O(1)` |
| `empty()` | 判断双端队列是否为空，若为空返回 `true`，否则返回 `false` | `O(1)` |
| `front()` | 返回队头元素，但不会删除该元素 | `O(1)` |
| `back()` | 返回队尾元素 | `O(1)` |
| `clear()` | 清空双端队列，使大小变为 0 | `O(n)` |

## 5. priority_queue


`priority_queue` 是优先队列，用于存储具有不同优先级的元素。默认情况下，它使用**最大堆**，即数值最大的元素优先出队。

它只维护堆顶元素 `top()`，队列中其他元素不保证整体有序。

### 5.1 初始化与自定义比较

默认大根堆：

```cpp
priority_queue<int> pq;
```

小根堆：

```cpp
priority_queue<int, vector<int>, greater<int>> pq;
```

lambda 比较器示例：

```cpp
auto cmp = [](const int& lhs, const int& rhs) {
    return lhs > rhs;
};

priority_queue<int, vector<int>, decltype(cmp)> pq(cmp);
```

对于 `priority_queue`，比较器的语义容易绕：上面这种 `lhs > rhs` 会让较小元素优先，即小根堆效果。

### 5.2 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `push(x)` | 将元素 `x` 推入队列中 | `O(log n)` |
| `pop()` | 移除队列中优先级最高的元素，这个函数不会返回被删除的元素 | `O(log n)` |
| `top()` | 返回队列中优先级最高的元素，但不删除该元素 | `O(1)` |
| `size()` | 返回队列中的元素个数 | `O(1)` |
| `empty()` | 判断队列是否为空，若为空返回 `true`，否则返回 `false` | `O(1)` |

### 5.3 注意事项

- `priority_queue` 不提供迭代器，无法用 `begin()` / `end()` 或范围 for 遍历。
- 不能直接修改优先队列中的元素。
- 大多数情况下，调用 `top()` 后会紧接着调用 `pop()`。
- 调用 `top()` 或 `pop()` 前，应确保优先队列非空。

## 6. map 与 unordered_map


`map` 是基于红黑树的关联容器，支持快速插入、查找、删除，并保持内部元素按 key 有序。每个元素由一个键和一个值组成。

### 6.1 初始化

```cpp
map<T_key, T_value> mp;
map<int, int> mp_int;
```

在 `map` 中，每个 key 唯一，对应一个 value。不同 key 可以对应相同 value。

如果使用自定义结构体作为 key，需要重载小于号 `<` 或提供比较器。

### 6.2 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `insert({key, value})` | 向 `map` 中插入一个键值对 `<key, value>` | `O(log n)` |
| `erase(key)` | 删除 `map` 中指定的键值对 | `O(log n)` |
| `find(key)` | 查找 `map` 中指定键对应的键值对的迭代器 | `O(log n)` |
| `operator[key]` | 查找 `map` 中指定键对应的值 | `O(log n)` |
| `count(key)` | 查找 `map` 中键的数量，由于键唯一，故只返回 0 或 1 | `O(log n)` |
| `size()` | 返回 `map` 中键值对的数量 | `O(1)` |
| `clear()` | 清空 `map` 中的所有键值对 | `O(n)` |
| `empty()` | 判断 `map` 是否为空 | `O(1)` |
| `begin()` | 返回 `map` 中第一个键值对的迭代器 | `O(1)` |
| `end()` | 返回 `map` 中最后一个键值对的下一个迭代器 | `O(1)` |

### 6.3 `[]` 运算符注意事项

注意：执行取值运算时要确保键存在。更准确地说：`mp[key]` 在 key 不存在时会自动插入一个默认值。

如果只是想判断是否存在，建议使用：

```cpp
if (mp.find(key) != mp.end()) {
    cout << mp[key] << '\n';
}
```

### 6.4 unordered_map

`unordered_map` 基于哈希表而不是红黑树。平均复杂度接近 `O(1)`，但特殊情况下可能退化到 `O(n)`。除非题目明确要求或性能瓶颈明显，普通 `map` 通常也足够；需要频繁计数/查找时，`unordered_map` 很常用。

## 7. set、multiset 与 unordered_set

### 7.1 set


`set` 是关联容器，底层通常是红黑树。集合中的元素唯一，并按一定顺序存储。默认升序。

初始化：

```cpp
set<T> mySet;
```

自定义比较器示例：

```cpp
struct cmp
{
    bool operator() (const int& u, const int& v) const
    {
        return u > v;
    }
};

set<int, cmp> st;
```

注意：`set` 的比较器应该返回 `bool`，用于定义严格弱序。把比较器理解为“映射到一个能直接比大小的类型”更接近哈希；如果要用哈希集合，应使用 `unordered_set` 并自定义 hash。

#### set 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `insert(x)` | 将元素插入集合中 | `O(log n)` |
| `erase(it or val)` | 从集合中删除指定元素，可以传入一个迭代器或一个值 | `O(log n)` |
| `clear()` | 清空集合中的所有元素 | `O(n)` |
| `find(x)` | 返回 `x` 在集合中的位置（迭代器），如果 `x` 不存在，则返回 `end()` | `O(log n)` |
| `count(x)` | 返回 `x` 在集合中出现的次数，返回值仅为 0 或 1 | `O(log n)` |
| `size()` | 返回集合中的元素个数 | `O(1)` |
| `empty()` | 判断集合是否为空，若为空返回 `true`，否则返回 `false` | `O(1)` |
| `begin()` | 返回指向集合中第一个元素的迭代器 | `O(1)` |
| `end()` | 返回指向集合中最后一个元素的下一个位置的迭代器 | `O(1)` |

调用 `erase(it)` 时，必须确保迭代器有效。

### 7.2 unordered_set

`unordered_set` 与 `unordered_map` 类似，基于哈希表，不保持有序，平均复杂度接近 `O(1)`。

### 7.3 multiset


`multiset` 是允许存储重复元素的 `set`。

#### multiset 常用操作表

| 方法 | 作用 | 时间复杂度 |
|---|---|---|
| `insert(x)` | 将元素 `x` 插入多重集合 | `O(log n)` |
| `erase(it or val)` | 从多重集合中删除一个或多个元素 | `O(k + log n)`，其中 `k` 为删除的元素个数 |
| `find(x)` | 在多重集合中查找元素 `x`，返回迭代器 | `O(log n)` |
| `count(x)` | 返回 `x` 在多重集合中出现的次数 | `O(log n)` |
| `size()` | 返回多重集合中元素个数 | `O(1)` |
| `empty()` | 判断多重集合是否为空 | `O(1)` |
| `clear()` | 清空多重集合中的所有元素 | `O(n)` |
| `begin()` | 返回指向多重集合中第一个元素位置的迭代器 | `O(1)` |
| `end()` | 返回指向多重集合中最后一个元素的下一个位置的迭代器 | `O(1)` |

删除时注意：

```cpp
ms.erase(x); // 删除所有等于 x 的元素
```

如果只想删除一个：

```cpp
auto it = ms.find(x);
if (it != ms.end()) ms.erase(it);
```

## 8. bitset


`bitset` 用于处理固定大小的位序列，即一串二进制位。可以把它看作一个长度可观的二进制数。初始化后各位默认均为 0。

注意：`bitset` 的索引从右向左计算，以保证与二进制表达方式一致。

### 8.1 初始化

```cpp
bitset<8> b1;                  // 00000000
bitset<8> b2(5);               // 00000101
bitset<8> b3(string("101"));   // 00000101
```

### 8.2 常用操作表

| 方法/运算 | 作用 | 时间复杂度 |
|---|---|---|
| `operator[idx]` | 返回 `bitset` 中第 `idx` 位的引用 | `O(1)` |
| `reset()` | 将 `bitset` 中所有位都设置为 0 | `O(n)` |
| `size()` | 返回 `bitset` 的大小，即位数 | `O(1)` |
| `count()` | 返回 `bitset` 中 1 的个数 | `O(n)` |
| `&`、`|`、`~`、`<<`、`>>` | 整体位运算 | `O(n)` |

常用方法：

| 方法 | 作用 |
|---|---|
| `set()` | 全部置 1 |
| `set(i)` | 第 `i` 位置 1 |
| `reset(i)` | 第 `i` 位置 0 |
| `flip()` | 全部翻转 |
| `flip(i)` | 第 `i` 位翻转 |
| `any()` | 是否存在 1 |
| `none()` | 是否全为 0 |
| `all()` | 是否全为 1 |
| `to_string()` | 转为字符串 |

虽然部分操作理论复杂度为 `O(n)`，但由于 `bitset` 内部优化及固定小常数，实际很快，可近似理解为 `O(n / w)`，其中 `w` 是机器字长。

## 9. string


`string` 用于表示和处理字符串，提供连接、查找、替换、插入、删除、子串等操作。可以把它想象成动态字符数组。

### 9.1 初始化

```cpp
string str = "Hello World";
string str1 = "Hello";
string str2 = str1;
char arr[] = "Hello";
string str3(arr);
string str4(5, 'A'); // "AAAAA"
```

### 9.2 常用操作表

| 方法/运算 | 作用 | 时间复杂度 |
|---|---|---|
| `length()` | 返回字符串长度 | `O(1)` |
| `empty()` | 判断字符串是否为空 | `O(1)` |
| `substr(pos, len)` | 返回从 `pos` 开始、长度为 `len` 的子串；若省略 `len`，截取到末尾 | `O(len)` |
| `erase(pos, len)` | 删除从 `pos` 开始、长度为 `len` 的子串 | `O(len)` |
| `operator + char/str` | 将字符或字符串拼接到当前字符串后面 | `O(len)` |
| `operator <, <=, >, >=, ==` | 按字典顺序比较字符串 | `O(n)` |
| `find(str)` | 返回指定字符串或字符在当前字符串中第一次出现的位置 | `O(nm)` |

常用方法：

| 方法 | 作用 |
|---|---|
| `size()` | 返回字符串长度，常与 `length()` 等价 |
| `push_back(c)` | 尾部加入字符 |
| `pop_back()` | 删除尾部字符 |
| `front()` / `back()` | 访问首尾字符 |
| `insert(pos, str)` | 在指定位置插入字符串 |
| `replace(pos, len, str)` | 替换指定区间 |
| `c_str()` | 获取 C 风格字符串指针 |

实践上，若需频繁修改或构造字符串，可考虑字符数组或预留容量，因为某些操作会更高效。现代 C++ 中也可使用：

```cpp
s.reserve(n);
```

减少扩容次数。

## 10. pair


`pair` 是模板类，用于存储一对值，包含两个公有成员变量：`first` 和 `second`。

| 用法 | 示例 |
|---|---|
| 定义有序对 | `pair<int, string> p;` |
| 初始化 | `pair<int, int> p = make_pair(1, 2);` |
| 访问值 | `int x = p.first; string s = p.second;` |
| 比较有序对 | 默认先比较 `first`，相等再比较 `second` |
| 作为返回值 | `pair<int, int> findMaxMin(...)` |
| 作为容器元素 | `vector<pair<int, int>> v;` |

也可以使用列表初始化：

```cpp
pair<int, int> p = {1, 2};
```
