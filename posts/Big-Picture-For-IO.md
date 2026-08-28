---
title: Big Picture For IO
tags:
  - IO
  - BigPicture
categories:
  - Operating System
date: 2026-08-15 12:07:00
katex: true
---

# The Big Picture


![1000](../assets/read-keyboard-block-wakeup-dark.png)

# 为什么先使用内核输入缓冲区（键盘缓冲区）

进程使用的是虚拟地址，但对于设备得到的是物理地址。设备不能把 P 的虚拟地址直接当成稳定的物理写入位置。

内核输入缓冲区不依赖 P 当前是否运行，可以先保存异步到达的数据。P 重新获得 CPU 后，内核才在 P 的进程上下文中检查并访问其用户地址，把数据交付给它。