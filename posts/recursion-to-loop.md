---
title: recursion to loop
tags:
  - Stack
  - InstructionSystem
categories:
  - Data Structure & Algorithm
date: 2026-04-15 00:00:00
katex: true
---

由[[Function-Calls-And-Stack-Frames#标准函数调用过程|汇编代码的函数栈帧切换机制]]所启发，得到一种将任意递归代码转化为利用栈的循环形式。

# 1. 汇编代码的函数栈帧切换机制

在调用函数时，机器会将当前指令指针（IP）以及调用者函数的入口地址(bp)压入栈中进行保存。在子函数返回时，机器就会从栈中弹出这两个值。又由于根据bp的值可以找到调用者函数的传入参数/内部变量的存储地址，所以从而恢复调用者函数的执行状态。

而对于返回值，则是会分配寄存器%eax专门地进行存储。

# 2. 伪代码

于是，对于任意递归代码，都可以通过栈来模拟递归的过程。且栈的元素就是函数的栈帧，它包含了函数的参数、内部变量、IP等信息, 伪代码定义如下：

```
struct StackFrame{
    函数的参数、内部变量
    int ip;
}
```

于是对于任一递归函数`r`

```text
func r(p){
    if(condition){
        return ret;
    }
    r(f1(p));
    ...
    r(f2(p));
    ...
    r(fn(p));
}
```

都有`l`与之等价：

```text
func l(p){
    type ret = 0;// 存返回值（如有）
    frame stack = nullset;
    stack.push((frame){p,0});
    while(!stack.empty()){
        cur = stack.top();
        //如果原递归函数有返回值，则出栈后将这个返回值传递给ret
        if(cur.condition){
            ...
            stack.pop();
            continue;
        }
        //ip的取值与原递归函数有几个递归点有关
        //若有n个递归点，则取值有n+1个, {ip_0 ... ip_n}
        //分别表示：未进入第一个递归点 进入/结束了第一个但未进入第二个 ... 进入/结束了第n-1个但未进入第n个 进入/结束了第n个
        switch(cur.ip){
            case 0 :
                //递归点1入栈
                ...
                stack.push(frame(f1(p),0));
                cur.ip=1;
            case 1:
                //可能还需要将ret传递给递归点1，2入栈
                ...
                stack.push(frame(fn(p),0));
                cur.ip=2;
            ...
            case n-1:
                //可能还需要将ret传递给递归点n-1, n入栈
                ...
                stack.push(frame(fn(p),0));
                cur.ip=n;
            default:
                //可能还需要将ret传递给递归点n, 结束
                ...
                stack.pop();
        }
    }
}
```

# 3. 示例

## 3.1 hanoi

递归写法：

```C
///input: src, tmp, dst, total plates
void hanoi(int src, int tmp, int dst, int n){
    if(n==0){
        return;
    }
    static int step = 0;
    hanoi(src,dst,tmp,n-1);
    printf("%d->%d, step %d, plate #%d\n",src, dst, ++step, n);
    hanoi(tmp, src, dst, n - 1);
    return;
}
```

则利用栈的循环写法:

```C
typedef struct {
    int src;
    int tmp;
    int dst;
    int n;
    int ip; // =0/1/2 未进入递归点 进入/结束第一个未进入第二个 进入/结束第二个
} Frame;

void hanoi(int src, int tmp, int dst, int n) {
    Frame stack[128];
    int top = -1;
    int step = 0;

    stack[++top] = (Frame){src, tmp, dst, n, 0};

    while (top != -1) {
        Frame *cur = &stack[top];
        if (cur->n == 0) {
            --top;
            continue;
        }
        if (cur->ip == 0) {
            cur->ip = 1;
            stack[++top] = (Frame){cur->src, cur->dst, cur->tmp, cur->n - 1, 0};
            continue;
        }
        if (cur->ip == 1) {
            printf("%d->%d, step %d, plate #%d\n", cur->src, cur->dst, ++step, cur->n);
            cur->ip = 2;
            stack[++top] = (Frame){cur->tmp, cur->src, cur->dst, cur->n - 1, 0};
            continue;
        }
        --top;
    }

    return;
}
```

## 3.2 postorder traversal

递归写法：
```C
///input: root
void postorder(TreeNode *root){
    if(root==NULL){
        return;
    }
    postorder(root->left);
    postorder(root->right);
    printf("->%d",root->val);
    return;
}
```
利用栈的循环写法：
```C
typedef struct {
    TreeNode *root;
    int ip; // =0/1/2 未进入递归点 进入/结束第一个未进入第二个 进入/结束第二个
} Frame;

void postorder(TreeNode *root){
    Frame stack[128];
    int top =-1;
    stack[++top] = (Frame){root, 0};
    while(top != -1){
        Frame *cur = &stack[top];
        if(cur->root == NULL){
            --top;
            continue;
        }
        if(cur->ip == 0){
            cur->ip = 1;
            stack[++top] = (Frame){cur->root->left, 0};
            continue;
        }
        if(cur->ip == 1){
            cur->ip = 2;
            stack[++top] = (Frame){cur->root->right, 0};
            continue;
        }
        printf("->%d", cur->root->val);
        --top;
    }
}
```

## 3.3 fibonacci

递归写法：
```C
int fib(int n){
    if(n<=1){
        return n;
    }
    return fib(n-1)+fib(n-2);
}
```

利用栈的循环写法：
```C
typedef struct{
    int n;
    int a;
    int b;
    int ip; //=0/1/2
} frame;

int fib(int n){
    int ret = 0; // 存返回值
    frame stack[128];
    int top = -1;
    stack[++top]=(frame){n,0,0,0};
    while(top!=-1){
        frame *f = &stack[top];
        if(f->n<=1){
            ret = f->n;
            --top;
            continue;
        }
        if(f->ip==0){
            f->ip = 1;
            stack[++top] = (frame){f->n-1,0,0,0};
            continue;
        }
        if(f->ip==1){
            f->a = ret;
            f->ip = 2;
            stack[++top] = (frame){f->n-2,0,0,0};
            continue;
        }
        if(f->ip==2){
            f->b = ret;
            ret = f->a+f->b;
            --top;
        }
    }
    return ret;
}

```
