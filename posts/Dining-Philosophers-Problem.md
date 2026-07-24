---
title: Dining Philosophers Problem
tags:
  - ProcessAndThread
categories:
  - Operating System
date: 2026-07-06 15:48:55
katex: true
---

# 问题模型

哲学家就餐问题描述的是 5 个哲学家围坐在圆桌旁。每两个哲学家之间有一支筷子。哲学家不断在“思考”和“进餐”之间切换：

- 思考不需要共享资源。
- 进餐必须同时拿到左、右两支筷子。
- 每支筷子是互斥资源，同一时刻只能被一个哲学家持有。
- 如果需要的筷子已经被别人持有，哲学家就等待。
- 进餐结束后，哲学家释放两支筷子，再继续思考。

![900](../assets/dining-philosophers-resource-ring.svg)

编号约定：

- 哲学家编号为 `0` 到 `4`。
- 筷子编号为 `0` 到 `4`。
- 哲学家 `i` 左边的筷子是 `chopstick[i]`。
- 哲学家 `i` 右边的筷子是 `chopstick[(i + 1) % 5]`。

这个问题只有互斥关系，但每个进程不是只申请一个临界资源，而是需要同时持有两个临界资源。

# 朴素写法与死锁

最直接的写法是每个哲学家先拿左筷子，再拿右筷子：

```text
semaphore chopstick[5] = {1, 1, 1, 1, 1};

Pi 进程：
while (1) {
    思考;

    P(chopstick[i]);           // 拿左筷子
    P(chopstick[(i + 1) % 5]); // 拿右筷子

    进餐;

    V(chopstick[i]);           // 放左筷子
    V(chopstick[(i + 1) % 5]); // 放右筷子
}
```

[html-card height=720](../assets/dining-philosophers-deadlock-slides.html)

如果 5 个哲学家同时拿起左筷子，就会出现：

| 哲学家 | 已持有 | 正在等待 |
| --- | --- | --- |
| `P0` | `C0` | `C1` |
| `P1` | `C1` | `C2` |
| `P2` | `C2` | `C3` |
| `P3` | `C3` | `C4` |
| `P4` | `C4` | `C0` |

每个人都占有一支筷子，又等待另一支筷子。等待关系首尾相接，没人能先吃完并释放资源，于是形成死锁。

# 三种破局方式

![900](../assets/dining-philosophers-solutions.svg)

## 限制最多 4 人竞争

增加一个计数信号量，限制同一时刻最多只有 4 个哲学家进入拿筷子的阶段：

```text
semaphore chopstick[5] = {1, 1, 1, 1, 1};
semaphore room = 4;

Pi 进程：
while (1) {
    思考;

    P(room);
    P(chopstick[i]);
    P(chopstick[(i + 1) % 5]);

    进餐;

    V(chopstick[i]);
    V(chopstick[(i + 1) % 5]);
    V(room);
}
```

只允许 4 个人竞争时，至少有一位哲学家没有持有筷子。这样不可能出现“5 个人各持一支筷子并等待下一支”的完整循环。

## 改变拿筷子顺序

让相邻哲学家的第一支筷子请求方向不同。例如：

- 奇数号哲学家先拿左筷子，再拿右筷子。
- 偶数号哲学家先拿右筷子，再拿左筷子。

```text
semaphore chopstick[5] = {1, 1, 1, 1, 1};

Pi 进程：
while (1) {
    思考;

    if (i 是奇数) {
        P(chopstick[i]);
        P(chopstick[(i + 1) % 5]);
    } else {
        P(chopstick[(i + 1) % 5]);
        P(chopstick[i]);
    }

    进餐;

    V(chopstick[i]);
    V(chopstick[(i + 1) % 5]);
}
```

朴素写法的问题是所有哲学家都沿同一方向请求资源，容易形成闭环。奇偶顺序不同后，相邻哲学家对同一支筷子的第一步请求会发生竞争，其中一个会直接阻塞，避免“大家都先拿到一支”的局面。

## 互斥执行拿筷子动作

再加一个 `mutex`，让“拿左筷子、拿右筷子”这一组动作互斥执行：

```text
semaphore chopstick[5] = {1, 1, 1, 1, 1};
semaphore mutex = 1;

Pi 进程：
while (1) {
    思考;

    P(mutex);
    {
        P(chopstick[i]);
        P(chopstick[(i + 1) % 5]);
    }
    V(mutex);

    进餐;

    V(chopstick[i]);
    V(chopstick[(i + 1) % 5]);
}
```

这里的 `mutex` 保护的不是筷子本身，而是“尝试拿两支筷子”这一段动作。它保证不会有多个哲学家交错执行拿筷子过程，因此不会出现五个人同时各持一支筷子的局面。

不过这种方法会降低并发度。即使某个哲学家左右两支筷子都空闲，他也可能因为别的哲学家正在执行拿筷子动作而暂时不能尝试。

哲学家就餐问题的核心不是“筷子怎么编号”，而是暴露了一类危险：多个进程逐个申请多个互斥资源时，可能因为资源分配顺序不当而全部阻塞。这个危险会在 [[Deadlock-Basics-And-Necessary-Conditions]] 中继续展开。
