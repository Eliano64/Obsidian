---
title: Function Calls And Stack Frames
tags:
  - ComputerOrganization
  - InstructionSystem
categories:
  - Computer Organization
date: 2026-07-02 11:39:24
katex: true
---


函数调用在机器级的工作是：

1. 从调用者跳转到被调用函数的第一条指令。
2. 被调用函数结束后，能回到调用点后面的那条指令，并继续使用调用者原来的执行状态。

因此，一次函数调用必须保存返回地址、保存必要的上下文、传递参数、提供局部变量空间，并把返回值交回调用者。

# 函数调用栈

运行中的进程有自己的虚拟地址空间。用户栈通常位于高地址区域，并向低地址增长。每发生一次函数调用，就会在栈上形成一个新的**栈帧**；函数返回时，这个栈帧被撤销。

![](../assets/function-stack-address-space.svg)

栈帧可以理解为某一次函数调用的“现场”：

- 参数：调用者传给被调用者的数据。
- 返回地址：被调用函数结束后应该回到哪里。
- 保存的旧 `ebp`：用于恢复调用者栈帧。
- 局部变量：被调用函数自己的临时数据。
- 必要时保存的寄存器值：避免调用其他函数时破坏中间结果。

递归调用并不特殊。每递归一次，就多压入一层栈帧。不同递归层的参数和局部变量互不覆盖，因为它们在不同的栈帧里。

# esp 与 ebp

在 32 位 x86 的典型栈帧模型中：

- `esp` 指向当前栈顶，执行 `push`、`pop`、`call`、`ret` 时会变化。
- `ebp` 指向当前栈帧的基地址，函数体内通常保持不变。

因为 `esp` 会随压栈、出栈改变，所以直接用 `esp` 访问参数和局部变量不稳定。建立栈帧后，参数和局部变量通常通过 `ebp` 加偏移访问。

![](../assets/x86-stack-frame-layout.svg)

典型偏移关系如下：

| 位置 | 含义 |
|---|---|
| `[ebp+12]` | 第 2 个参数 |
| `[ebp+8]` | 第 1 个参数 |
| `[ebp+4]` | 返回地址 |
| `[ebp]` | 保存的上一层 `ebp` |
| `[ebp-4]` | 第 1 个局部变量 |
| `[ebp-8]` | 第 2 个局部变量 |

> [!note] 为什么第 1 个参数是 `[ebp+8]`
> `call` 会先压入返回地址。被调用函数进入后又执行 `push ebp` 保存旧 `ebp`，再执行 `mov ebp, esp`。所以当前 `ebp` 指向保存的旧 `ebp`，`[ebp+4]` 是返回地址，`[ebp+8]` 才是第 1 个参数。

# push 与 pop

32 位 x86 中，一次普通压栈、出栈通常以 4 字节为单位。

- `push` 是先移动栈顶，再写入新栈顶。


```asm
esp = esp - 4
[esp] = x
```


- `pop` 是先读出旧栈顶，再移动栈顶。

```asm
x = [esp]
esp = esp + 4
```



这正好对应栈向低地址增长：压栈会让 `esp` 变小，出栈会让 `esp` 变大。

# call 与 ret

普通跳转指令只改变程序计数器；函数调用还必须保存返回地址。x86 中的程序计数器常称为 `eip`。

`call f` 做两件事：

1. 把返回地址压入(push)栈顶。**返回地址是 `call` 后面那条指令的地址。**
2. 把 `eip` 改为函数 `f` 的入口地址。

`ret` 也做两件事：

1. 从栈顶弹出(pop)返回地址。
2. 把这个地址写回 `eip`，使程序回到调用者。

[html-card height=720](../assets/call-ret-return-address-slides.html)

# 标准函数调用过程

下面以 `add(x, y)` 为例，采用常见的 32 位 x86 栈帧模型。

调用者准备参数：

```asm
push y          ; 第 2 个参数
push x          ; 第 1 个参数
call add        ; 压入返回地址，并跳转到 add
```

被调用者建立栈帧：

```asm
add:
  push ebp
  mov ebp, esp
  sub esp, local_size
```

这三条指令的含义是：

- `push ebp`：保存调用者的栈帧基准。
- `mov ebp, esp`：把当前栈顶设为本函数栈帧基准。
- `sub esp, local_size`：为局部变量预留空间。

被调用者返回前恢复栈帧：

```asm
  mov esp, ebp
  pop ebp
  ret
```

这三条指令的含义是：

- `mov esp, ebp`：丢弃局部变量区域，使栈顶回到保存旧 `ebp` 的位置。
- `pop ebp`：恢复调用者的 `ebp`。
- `ret`：弹出返回地址，恢复 `eip`。

[html-card height=1620](../assets/function-call-stack-frame-slides.html)

> [!note] `leave` 指令
> 有些汇编会用 `leave` 代替 `mov esp, ebp` 和 `pop ebp`。也就是说，`leave; ret` 常用于函数结尾。

> [!note] `enter` 指令
> `enter` 可用于建立栈帧，效果接近函数开头的 `push ebp` 和 `mov ebp, esp`。


## 参数、局部变量与返回值

建立栈帧后，函数体通常用固定偏移访问数据：

```asm
mov eax, [ebp+8]      ; 读取第 1 个参数 x
add eax, [ebp+12]     ; 加上第 2 个参数 y
mov [ebp-4], eax      ; 写入局部变量 sum
```

返回值通常放入 `eax`：

```asm
mov eax, [ebp-4]      ; 把返回值放入 eax
```

调用者在 `call` 返回后，就可以从 `eax` 取到返回值：

```asm
call add
mov [sum], eax
```

具体调用约定可能不同。例如参数由谁清理、哪些寄存器由调用者保存、哪些由被调用者保存，都属于调用约定的细节。但“返回地址压栈、栈帧保存现场、返回值经寄存器传回”是理解机器级函数调用的主线。

> [!note] 栈帧中的空闲区域
> 一些编译器会让栈帧大小按固定边界对齐，例如按 `16B` 的整数倍组织。这样栈帧中可能出现未使用的空闲区域，它不是新的变量，只是为了满足对齐或调用约定。

## 递归与栈帧

递归函数每调用一次自身，就形成一个新的栈帧。每一层栈帧都保存本层的参数、局部变量、返回地址和必要上下文。

手动模拟栈帧将任何递归改为循环详见[[../posts/recursion-to-loop|recursion-to-loop]]

# 小结

函数调用链：

```text
caller 准备参数
call 压入返回地址并跳转
callee 保存旧 ebp
callee 建立新 ebp
callee 分配局部变量
callee 通过 ebp 偏移访问参数和局部变量
callee 把返回值放入 eax
callee 恢复 esp 和 ebp
ret 弹出返回地址并恢复 eip
caller 使用 eax 中的返回值
```
