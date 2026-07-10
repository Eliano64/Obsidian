---
title: Expression Evaluation With Stack
tags:
  - DataStructureAndAlgorithm
  - Stack
categories:
  - Data Structure & Algorithm
date: 2026-06-27 15:24:00
katex: true
---

# 表达式求值

## 三种表达式

以二元运算符为例：

- 中缀表达式：运算符在两个操作数中间，如 `A + B`。
- 后缀表达式：运算符在两个操作数后面，如 `A B +`，也称逆波兰表达式。
- 前缀表达式：运算符在两个操作数前面，如 `+ A B`，也称波兰表达式。

计算机用栈处理后缀和前缀表达式很方便，因为运算符出现时，它需要的操作数已经在附近。

## 后缀表达式求值

[html-card](../assets/postfix-expression-evaluation.html)

从左到右扫描后缀表达式：

1. 遇到操作数，入栈。
2. 遇到运算符，弹出两个操作数。
3. 先弹出的是右操作数，后弹出的是左操作数。
4. 计算结果入栈。
5. 扫描结束后，栈顶就是表达式结果。

```c
int EvaluatePostfix(Token tokens[], int n) {
    SqStack stack;
    InitStack(&stack);

    for (int i = 0; i < n; i++) {
        if (tokens[i].kind == OPERAND) {
            Push(&stack, tokens[i].value);
        } else {
            int right, left;
            Pop(&stack, &right);
            Pop(&stack, &left);
            Push(&stack, Calculate(left, tokens[i].op, right));
        }
    }

    int result;
    Pop(&stack, &result);
    return result;
}
```

## 中缀转后缀

[html-card](../assets/infix-to-postfix.html)

从左到右扫描中缀表达式，用一个栈暂存还不能确定输出位置的运算符：

1. 遇到操作数：直接加入后缀表达式。
2. 遇到左括号：入栈。
3. 遇到右括号：弹出并输出运算符，直到弹出左括号；左右括号本身不输出。
4. 遇到运算符：弹出栈中优先级高于或等于当前运算符的运算符并输出，遇到左括号或栈空停止，再把当前运算符入栈。
5. 扫描结束：弹出栈中剩余运算符并输出。

同级运算符按从左到右结合时，遇到当前运算符要先弹出栈内优先级相同的运算符，这就是常说的左优先原则。

例如 `A*(B+C)-D/(E-F)` 的后缀表达式是 `ABC+*DEF-/-`。括号中的 `B+C` 和 `E-F` 虽然是低优先级运算，但右括号会触发括号内运算符先输出，使它们分别作为外层 `*` 和 `/` 的完整操作数。

```c
int Priority(char op) {
    if (op == '+' || op == '-') return 1;
    if (op == '*' || op == '/') return 2;
    return 0;
}

bool IsOperator(char ch) {
    return ch == '+' || ch == '-' || ch == '*' || ch == '/';
}

void InfixToPostfix(const char infix[], char postfix[]) {
    SqStack operatorStack;
    InitStack(&operatorStack);
    int outIndex = 0;

    for (int i = 0; infix[i] != '\0'; i++) {
        char token = infix[i];
        if (token == ' ') continue;

        if (isalnum((unsigned char)token)) {
            postfix[outIndex++] = token;
        } else if (token == '(') {
            Push(&operatorStack, token);
        } else if (token == ')') {
            char topOperator;
            while (GetTop(operatorStack, &topOperator) && topOperator != '(') {
                Pop(&operatorStack, &topOperator);
                postfix[outIndex++] = topOperator;
            }
            Pop(&operatorStack, &topOperator);  // 弹出 '('，但不输出
        } else if (IsOperator(token)) {
            char topOperator;
            while (GetTop(operatorStack, &topOperator) &&
                   topOperator != '(' &&
                   Priority(topOperator) >= Priority(token)) {
                Pop(&operatorStack, &topOperator);
                postfix[outIndex++] = topOperator;
            }
            Push(&operatorStack, token);
        }
    }

    char topOperator;
    while (Pop(&operatorStack, &topOperator)) {
        postfix[outIndex++] = topOperator;
    }
    postfix[outIndex] = '\0';
}
```

这段代码中的运算符栈只保存“还不能输出”的运算符。遇到当前运算符时，若栈顶运算符优先级更高，或优先级相同且按左结合处理，就先把栈顶运算符输出；否则当前运算符入栈，等待右操作数出现。

## 前缀表达式求值

前缀表达式通常从右向左扫描：

1. 遇到操作数，入栈。
2. 遇到运算符，弹出两个操作数。
3. 先弹出的是左操作数，后弹出的是右操作数。
4. 计算结果入栈。

前缀和后缀的关键差别在扫描方向和左右操作数顺序。

## 易错点

- 后缀求值中，先弹出的是右操作数。减法和除法不能写反。
- 中缀转后缀时，左括号只作为栈内边界，不进入后缀表达式。
- 一个中缀表达式若不规定结合规则，手算转换可能看起来不唯一；考试通常按运算符优先级和结合性，由左向右依次运算来确定结果。
