---
title: Parentheses Matching
tags:
  - DataStructureAndAlgorithm
  - Stack
categories:
  - Data Structure & Algorithm
date: 2026-06-27 15:24:00
katex: true
---

# 括号匹配

## 基本思路

括号匹配使用 [[stack-definition-and-operations|栈]] 保存尚未匹配的左括号。扫描字符串时：

1. 遇到左括号，入栈。
2. 遇到右括号，检查栈顶左括号是否匹配。
3. 匹配则弹出栈顶；不匹配则失败。
4. 扫描结束后，栈为空才表示全部匹配。

## 失败情况

括号匹配失败通常有三类：

- 扫描到右括号时栈空：右括号多了。
- 栈顶左括号与当前右括号类型不匹配：括号交叉或类型错误。
- 扫描结束后栈非空：左括号多了。

也可以记成三种错误：右括号单身、左右括号不匹配、左括号单身。

## C 代码

```c
bool IsMatchingPair(char left, char right) {
    return (left == '(' && right == ')') ||
           (left == '[' && right == ']') ||
           (left == '{' && right == '}');
}

bool CheckParentheses(const char text[]) {
    SqStack stack;
    InitStack(&stack);

    for (int i = 0; text[i] != '\0'; i++) {
        char ch = text[i];
        if (ch == '(' || ch == '[' || ch == '{') {
            Push(&stack, ch);
        } else if (ch == ')' || ch == ']' || ch == '}') {
            char left;
            if (!Pop(&stack, &left)) return false;
            if (!IsMatchingPair(left, ch)) return false;
        }
    }

    return StackEmpty(stack);
}
```

## 记忆点

左括号入栈，右括号只和最近的未匹配左括号比较。这个“最近”正好对应栈的后进先出性质。

括号匹配是栈应用的基础题型；表达式求值也依赖类似的“暂存尚未处理的符号”思想，见 [[expression-evaluation-with-stack|表达式求值]]。
