---
title: Monitor
tags:
  - OperatingSystem
  - ProcessAndThread
categories:
  - Operating System
date: 2026-07-06 17:12:00
katex: true
---


管程把共享数据和访问共享数据的过程封装在一起，并由语言或编译器负责保证互斥进入。程序员不再直接在每段临界区外手写 `P(mutex)` 和 `V(mutex)`，而是调用管程提供的入口过程。

![900](../assets/monitor-structure.svg)

# 管程

管程是一种特殊的软件模块，通常由四部分组成：

| 组成部分 | 含义 |
| --- | --- |
| 管程名 | 标识这一组共享数据和操作 |
| 局部共享数据结构 | 只能在管程内部被访问的数据，如缓冲区、计数器 |
| 一组过程 | 对共享数据进行操作的入口，如 `insert`、`remove` |
| 初始化语句 | 设置共享数据的初值 |

管程的基本特征有三条：

1. 局部于管程的数据只能被局部于管程的过程访问。
2. 进程只能通过调用管程内的过程进入管程，从而访问共享数据。
3. 每次只允许一个进程在管程内执行某个内部过程。

**封装共享数据**，**自动保证互斥进入**。

# 互斥进入

管程内部可以有多个入口过程，但同一时刻只允许一个进程在管程内执行。若 `P1` 正在执行 `insert`，`P2` 此时调用 `remove`，`P2` 也不能直接进入管程，只能在管程入口外等待。

这和普通互斥锁的区别在于：互斥规则不是散落在每个调用者代码中，而是成为管程本身的进入规则。只要共享数据只能从管程入口访问，互斥访问就被集中起来。

>[!note]
>管程解决的是“进入管程时的互斥”。如果进程进入管程后发现条件不满足，例如缓冲区空或缓冲区满，还需要条件变量解决“什么时候等待、什么时候唤醒”的同步问题。

# 条件变量

条件变量用于表示管程内部的等待条件。它不是资源数量计数器，而是**等待队列**。

常见操作有两个：

| 操作          | 含义                                |
| ----------- | --------------------------------- |
| `wait(c)`   | 当前进程因条件 `c` 不满足而阻塞，进入等待队列并释放管程使用权 |
| `signal(c)` | 唤醒一个等待在条件变量 `c` 上的进程；若无人等待，通常没有效果 |


[html-card height=720](../assets/monitor-condition-variable-flow-slides.html)

>[!warning] `wait/signal` 不是 `P/V` 的另一个名字
>信号量的 `P(S)` / `V(S)` 操作对象是信号量 `S`，重点是资源数量和等待队列；管程条件变量的 `wait(c)` / `signal(c)` 操作对象是条件变量 `c`，重点是条件不满足时让出管程和条件可能满足后唤醒等待者。
>
>
> | 对比点 | 信号量 `P/V` | 条件变量 `wait/signal` |
> | --- | --- | --- |
> | 操作对象 | 信号量 | 条件变量 |
> | 是否保存资源数量 | `S.value` 表示资源剩余数或等待情况 | 条件变量本身不表示资源数量 |
> | `P(S)` / `wait(c)` | `P(S)` 申请资源；资源不足则阻塞 | `wait(c)` 表示条件不满足而等待，并释放管程 |
> | `V(S)` / `signal(c)` | `V(S)` 释放资源，并可能唤醒等待者 | `signal(c)` 只唤醒等待在该条件上的进程 |
> | 无等待者时 | `V(S)` 通常会使资源数增加 | `signal(c)` 通常没有“积累一次唤醒”的效果 |
> | 互斥责任 | 需要程序员用互斥信号量保护临界区 | 管程入口规则已经保证互斥进入 |

因此，`notEmpty` 这种条件变量不是“产品数量”。产品数量仍由管程内部的 `count` 保存。`notEmpty` 只是“等待非空条件的队列”。当 `insert` 让 `count` 从 `0` 变为 `1` 后，`signal(notEmpty)` 的含义是：现在缓冲区可能非空，可以唤醒一个等产品的消费者。

# 生产者-消费者的管程表达

生产者-消费者问题里，共享数据是缓冲区。管程把缓冲区、计数器、入口过程和条件变量放在同一个模块中：

![900](../assets/monitor-producer-consumer-flow.svg)

可以把管程版伪代码写成：

```text
monitor ProducerConsumer
    buffer[n];
    count = 0;
    condition notFull, notEmpty;

    procedure insert(item)
        if count == n then
            wait(notFull);

        把 item 放入 buffer;
        count = count + 1;

        signal(notEmpty);

    procedure remove()
        if count == 0 then
            wait(notEmpty);

        从 buffer 取出 item;
        count = count - 1;

        signal(notFull);
        return item;
end monitor
```

外部生产者只调用 `insert(item)`，外部消费者只调用 `remove()`。缓冲区本身不暴露给外部进程。

这里有三层含义：

- 互斥由管程入口规则保证：同一时刻最多一个进程在 `insert` 或 `remove` 内执行。
- 缓冲区满时，生产者在 `notFull` 上等待，并释放管程。
- 缓冲区空时，消费者在 `notEmpty` 上等待，并释放管程。

`signal(notEmpty)` 表示“现在可能有产品了”，用于唤醒等待产品的消费者；`signal(notFull)` 表示“现在可能有空位了”，用于唤醒等待空位的生产者。
# Java synchronized

Java 的 `synchronized` 方法和代码块可以看作管程思想的一种体现：同一对象的 `synchronized` 方法同一时刻只能由一个线程执行。线程也可以使用 `wait()` 释放对象锁并等待，使用 `notify()` 或 `notifyAll()` 唤醒等待线程。

`synchronized` 对应互斥进入，`wait/notify` 对应条件等待和唤醒。
