---
title: Readers Writers Problem
tags:
  - ProcessAndThread
categories:
  - Operating System
date: 2026-07-06 14:59:18
katex: true
---

# 问题模型

读者-写者问题描述的是这样两类并发进程共享同一个文件或共享数据：

- 读者只读取共享数据，不改变数据内容。
- 写者会修改共享数据。
- 多个读者同时读同一份数据，不会产生副作用。
- 写者和读者同时访问，可能读到不一致的数据。
- 多个写者同时访问，可能互相覆盖写入结果。

因此它不是简单的所有进程互斥。真正的约束是**读读共享，读写互斥，写写互斥。**

![900](../assets/readers-writers-relation-matrix.svg)


# 三个基本量

读者-写者问题的关键是把**当前正在读的一组读者**看作一个整体。

| 名称      | 初值  | 含义                       |
| ------- | --- | ------------------------ |
| `rw`    | `1` | 共享文件访问权；写者独占，读者组整体占有     |
| `count` | `0` | 当前正在访问共享文件的读者数量          |
| `mutex` | `1` | 保护 `count` 的检查和修改，使其不可交错 |

`rw` 控制文件本身。写者写文件前执行 `P(rw)`，写完后执行 `V(rw)`。

`count` 记录读者组的人数。第一个读者进入时把 `rw` 锁住，最后一个读者退出时把 `rw` 释放。

`mutex` 不保护文件内容。它只保护 `count`。因为“检查 `count`、决定是否操作 `rw`、修改 `count`”必须连在一起完成。

# 读者优先写法

```text
semaphore rw = 1;       // 共享文件访问权
semaphore mutex = 1;    // 互斥访问 count
integer count = 0;      // 正在读的读者数

writer 进程：
	while (1) {
	    P(rw);
	    写文件;
	    V(rw);
	}

reader 进程：
	while (1) {
	    P(mutex);
	    {
	        if (count == 0) {
	            P(rw);
	        }
	        count = count + 1;
	    }
	    V(mutex);
	
	    读文件;
	
	    P(mutex);
	    {
	        count = count - 1;
	        if (count == 0) {
	            V(rw);
	        }
	    }
	    V(mutex);
	}
```

[html-card height=690](../assets/readers-writers-reader-flow-slides.html)

读者有三种情况：

- 第一个读者：发现进入前 `count == 0`，说明还没有读者占有共享文件，于是执行 `P(rw)`，阻止写者进入。
- 中间读者：进入前 `count > 0`，说明读者组已经占有共享文件，只需增加 `count`。
- 最后一个读者：退出后 `count == 0`，说明读者组已经全部离开，于是执行 `V(rw)`，允许写者进入。

读文件本身不放在 `mutex` 内。否则多个读者会被 `mutex` 人为串行化，读读共享就失效了。

##  count 必须互斥

[html-card height=690](../assets/readers-writers-count-race-slides.html)

如果没有 `mutex`，两个读者可能同时看到 `count == 0`，从而都把自己当作第一个读者。第一个读者执行 `P(rw)` 后，第二个读者再执行 `P(rw)` 就会被阻塞。结果是两个本来可以并发读取的读者，被错误地变成互斥关系。

退出时也一样。若两个读者并发修改 `count`，最后一个读者的判断可能丢失，导致 `rw` 没有被释放，写者永久等待。

因此 `mutex` 要包住两段代码，实现`count`互斥访问。

# 写者饥饿与公平闸门

上面的写法可能会导致**写者饥饿**：只要已经有读者在读，后续读者可以继续加入读者组；如果读者源源不断到来，写者可能一直等不到 `rw`。

一种改法是增加一个等待闸门 `w`。

| 名称  | 初值  | 含义                         |
| --- | --- | -------------------------- |
| `w` | `1` | 让读者和写者进入共享文件前先经过同一个入口 |

`w` 的作用不是保护共享文件，而是保护**排队入口**。
当写者到达并占住 `w` 后，后续读者不能继续插队进入读者组。已经进入读者组的读者仍然可以读完，最后一个读者释放 `rw` 后，等待中的写者就能继续执行。

![900](../assets/readers-writers-fairness-gate.svg)

伪代码：

```text
semaphore rw = 1;
semaphore mutex = 1;
semaphore w = 1;
integer count = 0;

writer 进程：
	while (1) {
	    P(w);
	    P(rw);
	    写文件;
	    V(rw);
	    V(w);
	}

reader 进程：
	while (1) {
	    P(w);
	    P(mutex);
	    {
	        if (count == 0) {
	            P(rw);
	        }
	        count = count + 1;
	    }
	    V(mutex);
	    V(w);
	
	    读文件;
	
	    P(mutex);
	    {
	        count = count - 1;
	        if (count == 0) {
	            V(rw);
	        }
	    }
	    V(mutex);
	}
```

这种写法通常称为读写公平法。它防止写者被不断到来的读者饿死。

