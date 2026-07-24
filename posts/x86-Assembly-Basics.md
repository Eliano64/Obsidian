---
title: x86 Assembly Basics
tags:
  - InstructionSystem
categories:
  - Computer Organization
date: 2026-07-02 11:40:41
katex: true
---


# 从高级语言到机器码

C/C++ 程序到可执行文件通常经历预处理、编译、汇编、链接几个阶段。其中与本章最相关的是：

![C 到机器码的层次](../assets/x86-code-translation-levels.svg)

- **高级语言**：面向人，描述算法逻辑，例如 `y = a * b + c`。
- **汇编语言**：用助记符描述机器指令，例如 `mov eax, ebx`。
- **机器语言**：CPU 真正执行的二进制编码。

汇编语言和机器语言大体一一对应。读汇编时重点看三件事：

1. 指令要做什么操作。
2. 操作数来自哪里。
3. 结果写回哪里，是否改变程序执行流。

# 基本格式


同一条 x86 指令可以有不同汇编格式：

| 项目    | Intel 格式                          | AT&T 格式          |
| ----- | --------------------------------- | ---------------- |
| 操作数顺序 | `op dst, src`                     | `op src, dst`    |
| 寄存器   | `eax`                             | `%eax`           |
| 立即数   | `5`                               | `$5`             |
| 主存地址  | `[ebx + 8]`                       | `8(%ebx)`        |
| 读写长度  | `byte ptr`、`word ptr`、`dword ptr` | 指令后缀 `b`、`w`、`l` |

例如同样是把 `ebx + ecx * 4 + 8` 所指地址处的 32 位数据读入 `eax`：

```asm
; Intel
mov eax, dword ptr [ebx + ecx * 4 + 8]

; AT&T
movl 8(%ebx, %ecx, 4), %eax
```

读材料或题目时先判断格式，再判断源和目的，避免把数据流方向看反。

笔记统一为 Intel 格式。
# 操作数位置

x86 指令中的操作数常来自三类位置：寄存器、主存、立即数。

![x86 操作数来源](../assets/x86-operand-forms.svg)

## 寄存器

常见 32 位寄存器：

| 类型 | 寄存器 | 常见用途 |
| --- | --- | --- |
| 通用寄存器 | `eax`、`ebx`、`ecx`、`edx` | 算术逻辑运算、临时数据 |
| 变址寄存器 | `esi`、`edi` | 字符串、数组、线性表处理 |
| 栈相关寄存器 | `esp`、`ebp` | 函数调用栈、栈帧定位 |
| 指令指针 | `eip` | 指向下一条将执行的指令 |

同一组通用寄存器还可以按更短位宽访问。例如：

| 32 位 | 16 位 | 高 8 位 | 低 8 位 |
| --- | --- | --- | --- |
| `eax` | `ax` | `ah` | `al` |
| `ebx` | `bx` | `bh` | `bl` |
| `ecx` | `cx` | `ch` | `cl` |
| `edx` | `dx` | `dh` | `dl` |

例如：

```asm
mov eax, ebx
mov ax, bx
mov ah, bl
```

这三条指令的读写宽度不同。

## 主存操作数

Intel 格式用中括号表示主存地址：

```asm
mov eax, dword ptr [af996h]
mov byte ptr [af996h], 5
mov eax, dword ptr [ebx + 8]
```

含义：

- `dword ptr [af996h]`：访问地址 `af996h` 处的 32 位数据。
- `byte ptr [af996h]`：访问地址 `af996h` 处的 8 位数据。
- `[ebx + 8]`：先计算地址 `ebx + 8`，再访问该地址处的数据。

常见读写长度：

| 写法 | 位数 | 含义 |
| --- | ---: | --- |
| `byte ptr` | `8` | 字节 |
| `word ptr` | `16` | 字 |
| `dword ptr` | `32` | 双字 |

## 立即数

立即数就是直接写在指令中的常量：

```asm
mov eax, 5
add eax, 10
```

立即数可以写成十进制，也可以写成十六进制。部分 Intel 风格材料用 `h` 后缀表示十六进制，例如 `af996h`。



# 常见数据处理指令

多数双操作数算术逻辑指令都遵循：

```asm
op d, s
```

即用 `d` 和 `s` 计算，结果写回 `d`。目的操作数不能是立即数。

| 功能 | 指令 | 含义 |
| --- | --- | --- |
| 加法 | `add d, s` | `d = d + s` |
| 减法 | `sub d, s` | `d = d - s` |
| 取负 | `neg d` | `d = -d` |
| 自增 | `inc d` | `d = d + 1` |
| 自减 | `dec d` | `d = d - 1` |
| 与 | `and d, s` | `d = d & s` |
| 或 | `or d, s` | `d = d \| s` |
| 异或 | `xor d, s` | `d = d ^ s` |
| 非 | `not d` | `d = ~d` |
| 逻辑左移 | `shl d, s` | `d` 左移 `s` 位 |
| 逻辑右移 | `shr d, s` | `d` 逻辑右移 `s` 位 |

乘除法在 x86 中有更强的隐含寄存器约定：

| 指令 | 含义 |
| --- | --- |
| `mul s` | 无符号乘法，常隐含使用 `eax`，乘积可能放入 `edx:eax` |
| `imul ...` | 有符号乘法，形式较多 |
| `div s` | 无符号除法，常把 `edx:eax` 作为被除数，商入 `eax`，余数入 `edx` |
| `idiv s` | 有符号除法，寄存器约定类似 |

乘除法比普通 `add/sub` 更依赖隐含寄存器，具体要求具体分析。

# 比较、标志位与条件转移

选择结构和循环结构的机器级表示通常依赖：

```asm
cmp a, b
jxxx label
```

`cmp a, b` 本质上做一次 `a - b`，但不保存差值，只把运算结果的状态写入标志寄存器。随后条件转移指令根据这些标志位决定是否跳转。

![cmp 与条件转移](../assets/x86-cmp-branch-flow.svg)

常见标志位：

| 标志位 | 名称 | 含义 |
| --- | --- | --- |
| `ZF` | Zero Flag | 结果为 0 时置 1 |
| `SF` | Sign Flag | 结果为负时置 1 |
| `CF` | Carry Flag | 无符号运算产生进位或借位时置 1 |
| `OF` | Overflow Flag | 有符号运算溢出时置 1 |

常见条件转移：

| 指令            | 典型含义           | 主要判断的标志位          |
| ------------- | -------------- | ----------------- |
| `je` / `jz`   | 相等或结果为 0 时跳转   | `ZF=1`            |
| `jne` / `jnz` | 不相等或结果不为 0 时跳转 | `ZF=0`            |
| `jg`          | 有符号大于时跳转       | `ZF=0` 且 `SF=OF`  |
| `jge`         | 有符号大于等于时跳转     | `SF=OF`           |
| `jl`          | 有符号小于时跳转       | `SF!=OF`          |
| `jle`         | 有符号小于等于时跳转     | `ZF=1` 或 `SF!=OF` |
| `ja`          | 无符号大于时跳转       | `CF=0` 且 `ZF=0`   |
| `jb`          | 无符号小于时跳转       | `CF=1`            |

> [!warning]
> 有符号比较和无符号比较使用的条件转移不同。要先确认比较对象按有符号数还是无符号数解释。

# 分支结构

无条件转移指令：

```asm
jmp label
```

表示直接把指令指针改为 `label` 对应的位置，类似高级语言中的无条件跳转。

条件分支常见结构：

[html-card height=620 step=80](../assets/x86-branch-structure-slides.html)

```asm
			   ; if (a > b) {
cmp a, b       
jle ELSE       
			   ;   if 部分
jmp END        ; if 部分执行完，跳过 else

ELSE:          
               ; } else {
               ;   else 部分
               
END:
                ; 分支结束
```


1. `cmp` 生成标志位。
2. 条件转移决定是否进入 `else`。
3. `if` 部分执行完后，通常用 `jmp` 跳过 `else`。
4. 两条路径最终在 `END` 汇合。

> [!warning] 选择结构的机器级代码只要求控制流等价，不要求代码块的物理顺序和 C 源码一致。
> `if (comp_A) then statement_B; else statement_C` 对应的汇编一定需要先计算 `comp_A`，也一定需要条件转移；**但 `statement_B` 的代码不一定排在 `statement_C` 前面。**
>
> 也可以把 `else` 部分放在顺序执行位置，把 `if` 部分放在后面：
> ```asm
> 			   ; if (a > b) {
> cmp a, b
> jg IF_PART     ; 若 a > b，跳到 if 部分
>                ;   else 部分
> jmp END
>
> IF_PART:
>                ;   if 部分
> END:
>                ; 分支结束
>```
> 
> 因此，判断选择结构的机器级代码时，要看跳转目标和控制流。


> [!example] 若想要汇编代码块物理顺序与 C 源码一致，条件跳转可写为 C 条件的反条件，用来跳过 `if` 部分进入 `ELSE` 分支。
> 例如 `if (a > b)` 可写成 `jle ELSE`；若源码是 `if (a <= b)`，可写成 `jg ELSE`。



# 循环结构

用条件转移实现循环时，通常有四个部分：

1. 循环前初始化。
2. 判断是否直接跳过循环。
3. 循环主体。
4. 判断是否继续循环。

典型形式：

```asm
				; for (int ecx = 0; ecx < 10; ecx++) {
mov ecx, 0      ; 初始化
LOOP_TOP:
	cmp ecx, 10     ; 判断是否还能进入本轮循环
	jge LOOP_END    ; 若 ecx >= 10，跳出循环
				;   循环主体
	inc ecx         ; ecx++
	jmp LOOP_TOP    ; 回到循环判断
LOOP_END:
				; }
```


[html-card height=640 step=80](../assets/x86-loop-structure-slides.html)


这类循环中，`cmp ecx, 10` 的**执行次数通常比循环主体多一次**。以上例为例，循环主体在 `ecx=0..9` 时执行 `10` 次；当 `ecx=10` 时，还要再执行一次 `cmp`，发现条件不满足后跳出，所以循环入口处的判断共执行 `11` 次。

## loop 指令

x86 也提供 `loop` 类指令，通常隐含使用 `ecx` 作为计数器：

```asm
LOOP_TOP:
    ; 循环主体
    loop LOOP_TOP
```

`loop label` 的典型效果可理解为：

```asm
ecx = ecx - 1
if (ecx != 0) goto label
```

因此，`loop label` 大致等价于：

```asm
dec ecx
cmp ecx, 0
jne label
```

常见变体还有 `loopz`、`loopnz`：

| 指令 | 继续循环条件 |
|---|---|
| `loop` | `ecx != 0` |
| `loopz` / `loope` | `ecx != 0` 且 `ZF = 1` |
| `loopnz` / `loopne` | `ecx != 0` 且 `ZF = 0` |

`loop` 指令能让计数循环更紧凑，但用普通条件转移也能表达同样的控制流。
