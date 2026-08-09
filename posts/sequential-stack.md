---
title: Sequential Stack
tags:
  - Stack
categories:
  - Data Structure & Algorithm
date: 2026-06-27 15:24:00
katex: true
---
# 定义

顺序栈用一组连续存储单元存放栈中元素，本质上是用数组实现 [[Stack-Definition-And-Operations|栈]]。

常见约定：`top` 指向栈顶元素。

```c
#define MaxSize 50

typedef struct {
    ElemType data[MaxSize];
    int top;
} SqStack;
```

初始化时令 `top = -1`，表示栈中没有元素。

```c
void InitStack(SqStack *stack) {
    stack->top = -1;
}
```

# 进栈

[html-card](../assets/sequential-stack-push-pop.html)

`Push(&S, x)` 的关键是先判断栈满，再移动 `top`，最后写入新元素：

```c
bool Push(SqStack *stack, ElemType value) {
    if (stack->top == MaxSize - 1) return false;
    stack->top++;
    stack->data[stack->top] = value;
    return true;
}
```

若 `top` 指向栈顶元素，则栈满条件是 `top == MaxSize - 1`。

> [! note] 上溢
> 定长顺序栈已满时仍要求进栈，称为**上溢**。这里的 `Push` 在写数组之前判满并返回 `false`，表示进栈失败；正确实现不会真的越过数组边界。

# 出栈

`Pop(&S, &x)` 的关键是先判断栈空，再取出栈顶元素，最后移动 `top`：

```c
bool Pop(SqStack *stack, ElemType *value) {
    if (stack->top == -1) return false;
    *value = stack->data[stack->top];
    stack->top--;
    return true;
}
```

栈空条件是 `top == -1`。

> [! note] 下溢
> 空栈仍要求出栈或读取栈顶，称为**下溢**。`Pop` 和 `GetTop` 都应先判空，再访问 `data[top]`。

# 读栈顶

```c
bool GetTop(SqStack stack, ElemType *value) {
    if (stack.top == -1) return false;
    *value = stack.data[stack.top];
    return true;
}
```

`GetTop` 不改变 `top`。这点要和 `Pop` 区分。

# 另一种 top 约定

 `top` 可以指向下一个入栈元素的位置，此时：

- 初始化：`top = 0`
- 栈空：`top == 0`
- 栈满：`top == MaxSize`
- 进栈：先写 `data[top] = x`，再 `top++`
- 出栈：先 `top--`，再读 `data[top]`

> [! tip] 两种约定的判断
> 栈为空时，若 `top` 指向数组第一个有效下标之前，则表示“栈顶元素下标”；若指向数组第一个有效下标，则表示“下一个入栈元素的下标”。

# 缺点

顺序栈容量固定。若数组大小确定后无法扩展，栈满时不能继续进栈。两个同类型栈需要共用空间时，可用 [[Shared-Stack|共享栈]] 提高空间利用率。
