---
title: Cigarette Smokers Problem
tags:
  - OperatingSystem
  - ProcessAndThread
categories:
  - Operating System
date: 2026-07-06 17:36:00
katex: true
---

# 问题模型

吸烟者问题有一个供应者进程和三个吸烟者进程。卷烟需要三种材料：烟草、纸、胶水。

- 吸烟者 1 自带烟草，需要纸和胶水。
- 吸烟者 2 自带纸，需要烟草和胶水。
- 吸烟者 3 自带胶水，需要烟草和纸。
- 供应者每次把两种材料组合放到桌上。
- 拥有第三种材料的吸烟者取走桌上的组合，卷烟并抽完。
- 吸烟者抽完后通知供应者，供应者再放下一组材料。

![900](../assets/cigarette-smokers-model.svg)

这个问题可以看成特殊的生产者-消费者问题：供应者是单生产者，三个吸烟者是三类消费者，桌子是容量为 1 的缓冲区。不同点在于，供应者生产的不是同一种产品，而是三种不同材料组合。

# 同步关系

桌上可能出现三种组合：

| 桌上组合 | 谁能取走 | 对应信号量 |
| --- | --- | --- |
| 纸 + 胶水 | 自带烟草的吸烟者 1 | `offer1` |
| 烟草 + 胶水 | 自带纸的吸烟者 2 | `offer2` |
| 烟草 + 纸 | 自带胶水的吸烟者 3 | `offer3` |

还需要一个 `finish` 信号量，表示某个吸烟者已经取走材料并抽完，供应者可以继续放下一组材料。

| 信号量 | 初值 | 含义 |
| --- | --- | --- |
| `offer1` | `0` | 桌上有组合 1：纸 + 胶水 |
| `offer2` | `0` | 桌上有组合 2：烟草 + 胶水 |
| `offer3` | `0` | 桌上有组合 3：烟草 + 纸 |
| `finish` | `0` | 吸烟者完成一次吸烟 |

这里的同步方向是“前 V 后 P”：

- 供应者先放组合，再 `V(offerX)` 通知对应吸烟者。
- 对应吸烟者执行 `P(offerX)`，拿到材料后卷烟、吸烟。
- 吸烟者完成后 `V(finish)`。
- 供应者执行 `P(finish)`，等完成信号后再进入下一轮。

[html-card height=720](../assets/cigarette-smokers-flow-slides.html)

# 伪代码

```text
semaphore offer1 = 0;    // 桌上有纸 + 胶水
semaphore offer2 = 0;    // 桌上有烟草 + 胶水
semaphore offer3 = 0;    // 桌上有烟草 + 纸
semaphore finish = 0;    // 吸烟者完成一次吸烟

int i = 0;               // 用于轮流提供三种组合

provider 进程：
    while (1) {
        if i == 0 then
            把纸和胶水放到桌上;
            V(offer1);
        else if i == 1 then
            把烟草和胶水放到桌上;
            V(offer2);
        else
            把烟草和纸放到桌上;
            V(offer3);

        i = (i + 1) mod 3;

        P(finish);
    }

smoker1 进程：
    while (1) {
        P(offer1);
        从桌上取走纸和胶水;
        卷烟并抽掉;
        V(finish);
    }

smoker2 进程：
    while (1) {
        P(offer2);
        从桌上取走烟草和胶水;
        卷烟并抽掉;
        V(finish);
    }

smoker3 进程：
    while (1) {
        P(offer3);
        从桌上取走烟草和纸;
        卷烟并抽掉;
        V(finish);
    }
```

# 为什么不需要单独的 mutex

桌子可以看成容量为 1 的缓冲区。供应者每次放入一个组合后，会立刻执行 `P(finish)` 等待吸烟者完成；在此之前不会继续放第二个组合。

对吸烟者来说，同一时刻 `offer1`、`offer2`、`offer3` 至多只有一个能通过 `P` 操作。只有对应组合的吸烟者能取走材料，其他吸烟者仍阻塞。

因此，桌面访问已经被三个 `offer` 和一个 `finish` 的同步关系串行化。这个特定模型下不需要额外设置 `mutex`。若题目允许供应者连续放多组材料，或允许多个进程同时操作同一个桌面结构，就要重新考虑互斥保护。