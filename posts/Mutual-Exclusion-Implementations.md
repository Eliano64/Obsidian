---
title: Mutual Exclusion Implementations
tags:
  - ProcessAndThread
categories:
  - Operating System
date: 2026-07-06 12:44:23
katex: true
---

# 互斥实现方法的读法

每一种方法都要看三件事：

1. **进入区做什么**：如何检查临界区是否可进入，如何表达“我要进入”。
2. **退出区做什么**：如何释放临界区，如何让其他进程继续。
3. **是否违反某原则**：空闲让进、忙则等待、有限等待、让权等待中哪条做不到。

软件方法完全依靠普通变量和程序执行顺序。硬件方法把某些关键动作做成不可中断的原子操作。

下面均以两个进程 $P_0$、$P_1$ 竞争同一个临界资源为例。

# 软件实现方法


| 方法 | 共享变量 | 进入区核心动作 | 退出区核心动作 | 主要问题 |
| --- | --- | --- | --- | --- |
| 单标志法 | `turn` | 只允许 `turn == i` 的进程进入 | 把 `turn` 改成对方编号 | 必须轮流进入，违反空闲让进 |
| 双标志先检查法 | `flag[2]` | 先检查对方意愿，再设置自己意愿 | 清除自己意愿 | 检查后、上锁前可被切换，违反忙则等待 |
| 双标志后检查法 | `flag[2]` | 先设置自己意愿，再检查对方意愿 | 清除自己意愿 | 双方都先上锁时会互相等待，违反空闲让进和有限等待 |
| Peterson 算法 | `flag[2]`、`turn` | 先表达意愿，再主动谦让，再检查 | 清除自己意愿 | 前三条原则满足，但仍忙等，违反让权等待 |

## 单标志法

单标志法使用一个共享变量 `turn`，表示当前允许哪个进程进入临界区。

```text
P0 进程：
while (turn != 0);
critical section;     // 临界区
turn = 1;             // 退出区
remainder section;    // 剩余区

P1 进程：
while (turn != 1);
critical section;     // 临界区
turn = 0;             // 退出区
remainder section;    // 剩余区
```

如果 `turn` 初值为 0，那么 $P_0$ 可以先进入。$P_0$ 退出时把 `turn` 改成 1，于是下一次只能 $P_1$ 进入。

它能保证同一时刻最多一个进程进入临界区，但代价是必须严格轮流。若 `turn == 0`，而 $P_0$ 暂时不想进入临界区，$P_1$ 即使想进入也只能等待。临界区明明空闲，却不允许请求者进入，所以违反**空闲让进**。

## 双标志先检查法

双标志先检查法用 `flag[i]` 表示 $P_i$ 是否想进入临界区。进入区先检查对方是否想进入；若对方不想进入，再把自己的 `flag` 置为 `true`。

```text
P0 进程：
while (flag[1]);
flag[0] = true;
critical section;     // 临界区
flag[0] = false;      // 退出区
remainder section;    // 剩余区

P1 进程：
while (flag[0]);
flag[1] = true;
critical section;     // 临界区
flag[1] = false;      // 退出区
remainder section;    // 剩余区
```


问题出在“检查”和“上锁”不是一个原子动作。

[html-card height=650](../assets/double-flag-precheck-interleaving-slides.html)


一种可能的交错执行顺序是：

1. $P_0$ 检查 `flag[1] == false`，通过检查。
2. 还没来得及执行 `flag[0] = true`，发生进程切换。
3. $P_1$ 检查 `flag[0] == false`，也通过检查。
4. 两个进程随后分别把自己的 `flag` 置为 `true`，并进入临界区。

因此，双标志先检查法可能让两个进程同时进入临界区，违反**忙则等待**。

## 双标志后检查法

双标志后检查法把顺序反过来：先进场声明“我要进入”，再检查对方是否也想进入。

```text
P0 进程：
flag[0] = true;
while (flag[1]);
critical section;     // 临界区
flag[0] = false;      // 退出区
remainder section;    // 剩余区

P1 进程：
flag[1] = true;
while (flag[0]);
critical section;     // 临界区
flag[1] = false;      // 退出区
remainder section;    // 剩余区
```

这种做法避免了“检查通过后还没上锁”的漏洞，但会出现另一个问题：双方都先把自己的 `flag` 置为 `true`。

[html-card height=650](../assets/double-flag-postcheck-interleaving-slides.html)

当 $P_0$ 设置 `flag[0] = true` 后被切换，$P_1$ 又设置 `flag[1] = true`。此后 $P_0$ 检查到 `flag[1] == true`，$P_1$ 检查到 `flag[0] == true`，双方都在等待对方清除标志。

此时临界区没有任何进程在使用，却没有进程能进入，所以违反**空闲让进**；若双方长期卡住，也违反**有限等待**。

## Peterson 算法

Peterson 算法把双标志法的“表达意愿”和单标志法的“谦让”结合起来。

```text
P0 进程：
flag[0] = true;
turn = 1;
while (flag[1] && turn == 1);
critical section;     // 临界区
flag[0] = false;      // 退出区
remainder section;    // 剩余区

P1 进程：
flag[1] = true;
turn = 0;
while (flag[0] && turn == 0);
critical section;     // 临界区
flag[1] = false;      // 退出区
remainder section;    // 剩余区
```

进入区的含义是：

- `flag[i] = true`：我想进入临界区。
- `turn = j`：如果对方也想进入，我愿意让对方先进入。
- `while (flag[j] && turn == j)`：对方确实想进入，而且最后一次谦让是我做出的，那我等待。

[html-card height=670](../assets/peterson-mutual-exclusion-reasoning-slides.html)


> [!important] 为什么Peterson算法能避免前面算法的问题？  
> Peterson 算法不会让两个进程同时进入，关键在于 `turn` 是单个共享变量。双方都想进入时，`flag[0]` 和 `flag[1]` 都为 `true`，但 `turn` 最终只能等于 0 或 1。
> - 若 `turn == 1`，$P_0$ 的等待条件 `flag[1] && turn == 1` 成立，$P_0$ 等待；$P_1$ 的等待条件 `flag[0] && turn == 0` 不成立，$P_1$ 进入。
> - 若 `turn == 0`，$P_1$ 等待，$P_0$ 进入。
> 因此，两个等待条件不可能同时为假，两个进程不可能同时越过进入区；也不可能同时为真，因为 `turn` 不可能同时等于 0 和 1。
>
> 若只有 $P_0$ 想进入，`flag[1] == false`，$P_0$ 不会等待，满足空闲让进。若 $P_0$、$P_1$ 都想进入，后写入 `turn` 的进程相当于最后一次表示谦让，它会等待；另一个进程可以进入。因此 Peterson 算法能满足空闲让进、忙则等待、有限等待。

但等待者仍然停在 `while` 循环中反复检查条件，占用 CPU，所以不满足**让权等待**。

# 硬件实现方法

软件方法的根本困难是：普通语句之间可能发生进程切换。硬件方法让某些关键指令在执行期间不可被中断，从而得到原子性。

## 中断屏蔽方法

中断屏蔽方法在进入临界区前关中断，退出临界区后开中断。

```text
关中断;
critical section;     // 临界区
开中断;
remainder section;    // 剩余区
```

关中断后，当前处理机不会因为中断而转入中断处理程序，也就不会在临界区中发生由中断引起的进程切换。这样能保证当前进程完整执行临界区。

这种方法简单、高效，但限制很明显：

- 不适用于多处理机。一个处理机关中断，并不能阻止其他处理机上的进程访问同一临界资源。
- 不适用于用户进程。开中断、关中断属于特权指令，不能让用户程序随意执行。

## TestAndSet / TSL 指令

TestAndSet 也称 TS、TSL 或 TestAndSetLock。它把“读取旧锁值”和“设置新锁值”合成一个不可分割的硬件动作。

![900](../assets/atomic-test-and-set-action.svg)

```text
TestAndSet(lock):
	old = lock;
	lock = true;
	return old;
```

用 TSL 实现互斥时，`lock == false` 表示临界区未上锁，`lock == true` 表示临界区已上锁。

```text
while (TestAndSet(lock))
	;
critical section;     // 临界区
lock = false;         // 退出区
remainder section;    // 剩余区
```

若 `lock` 原来是 `false`，TSL 返回 `false`，循环结束，进程进入临界区。同时 TSL 已经把 `lock` 改成 `true`，后续进程会被挡住。

若 `lock` 原来是 `true`，TSL 返回 `true`，进程继续在 `while` 中循环等待。

TSL 的优点是实现简单，并且适用于多处理机；缺点是等待者会持续占用 CPU，属于忙等，不满足让权等待。

## Swap / XCHG 指令

Swap 也称 Exchange 或 XCHG。它的原子动作是交换两个变量的值。

```text
Swap(a, b):
temp = a;
a = b;
b = temp;
```

用 Swap 实现互斥时，进程先准备一个局部变量 `key = true`，然后不断把 `key` 与共享锁变量 `lock` 交换。

```text
key = true;
while (key == true)
    Swap(lock, key);
critical section;     // 临界区
lock = false;         // 退出区
remainder section;    // 剩余区
```

若 `lock` 原来是 `false`，交换后 `key` 变成 `false`，循环结束；同时 `lock` 变成 `true`，临界区被上锁。若 `lock` 原来是 `true`，交换后 `key` 仍为 `true`，进程继续忙等。

从互斥效果看，Swap 与 TSL 的作用类似：都用硬件原子动作完成“检查并上锁”。因此它同样实现简单、适用于多处理机，也同样不满足让权等待。

# 互斥锁与自旋锁

互斥锁把“进入区申请锁、退出区释放锁”封装成`acquire(lock)`和`release(lock)`两个操作

```text
acquire(lock);
{
	critical section;     // 临界区
}
release(lock);
remainder section;    // 剩余区
```

如果 `acquire(lock)` 在锁已被占用时反复循环检查，而不是阻塞当前进程，这种锁就是**自旋锁**。TSL、Swap 这类硬件原语常用来实现自旋锁。

自旋锁的特点是：

- 等待期间不发生进程阻塞和唤醒，也就没有上下文切换开销。
- 等待者会一直占用 CPU，违反让权等待。
- 在多处理器系统中，如果临界区很短，一个处理器自旋等待，另一个处理器很快释放锁，代价可能低于阻塞和唤醒。
- 在单处理机系统中，自旋通常不合适。等待者占着处理机反复检查，而持锁进程无法运行并释放锁。

互斥锁不一定都采用忙等。若锁不可用时，系统让进程阻塞并把 CPU 让给其他进程，这类实现就符合让权等待；代价是需要进程切换、阻塞队列管理和唤醒。

# 方法对比

| 方法          | 能否保证互斥      | 是否让权等待 | 适用特点                  |
| ----------- | ----------- | ------ | --------------------- |
| 单标志法        | 能           | 否      | 逻辑简单，但强制轮流，资源空闲也可能进不去 |
| 双标志先检查法     | 不能稳定保证      | 否      | 检查和上锁之间可被切换           |
| 双标志后检查法     | 能避免同时进入     | 否      | 可能双方都等待，临界区空闲也进不去     |
| Peterson 算法 | 能           | 否      | 软件算法中较完整，但仍忙等         |
| 中断屏蔽        | 能，限单处理机内核场景 | 不适用    | 简单高效，但不能给用户进程使用       |
| TSL / Swap  | 能           | 否      | 原子检查并上锁，适合多处理机，但忙等    |
| 阻塞式互斥锁      | 能           | 是      | 等待时释放 CPU，但有上下文切换开销   |
