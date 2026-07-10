---
title: monotonic stack
tags:
  - DataStructureAndAlgorithm
  - Stack
categories:
  - Data Structure & Algorithm
date: 2025-07-21 00:00:00
katex: true
---

> 他向远方望去，无法看到高山背后的矮山，只看到一座座更高的山峰。————by [灵神](https://leetcode.cn/discuss/post/3579480/ti-dan-dan-diao-zhan-ju-xing-xi-lie-zi-d-u4hk/)

## 1. 总结

单调栈就是保持栈内元素单调的栈。**当新元素破坏单调性时，弹出栈顶元素，弹出的瞬间就找到栈顶元素对应的答案**。

栈里存放着暂时还没有找到对应答案的元素。新元素入栈时，如果栈顶元素使得栈单调性被破环，那么栈顶元素的答案产生，栈顶元素找到答案，出栈。因此需要建立的栈的单调性与题目的答案需求往往是“反过来”。要下一个更大就要单减；要下一个更小就要单增。

```txt
algorithm generalized-monotonic-stack(A, cmp, process)
    stack S ← ∅
    for i ← 0 to n-1 do:
        while S ≠ ∅ and condition(A[i], A[S.top()]) do: // 栈的单调性被破坏，while是为了让栈里满足条件的元素都可以出栈
            j ← S.pop() // 栈顶出栈
            recordAnswer(j, i)  // 计算并记录栈顶对应的答案
        end while
        S.push(i) // 栈为空或者栈的单调性没有被破坏，则新元素一定不是某个元素的答案，入栈
    end for
end algorithm
```

### 时间复杂度为什么是 O(n)？

每个元素最多入栈一次，出栈一次，总操作数 ≤ 2n。

## 2. 问题分类

### 类型1：找下一个更大/更小元素

**模板：**

```cpp
// 下一个更大元素
while (!st.empty() && arr[i] > arr[st.top()]) {
    result[st.top()] = arr[i];
    st.pop();
}

// 下一个更小元素  
while (!st.empty() && arr[i] < arr[st.top()]) {
    result[st.top()] = arr[i];
    st.pop();
}
```

### 类型2：计算面积/距离

**模板：**

```cpp
// 柱状图最大矩形
while (!st.empty() && heights[i] < heights[st.top()]) {
    int h = heights[st.top()];
    st.pop();
    int w = st.empty() ? i : i - st.top() - 1;
    maxArea = max(maxArea, h * w);
}

// 每日温度（计算距离）
while (!st.empty() && temps[i] > temps[st.top()]) {
    int j = st.top();
    st.pop();
    result[j] = i - j;  // 距离
}
```

## 3. 典型例题

### 例1：下一个更大元素

```cpp
vector<int> nextGreaterElement(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, -1);
    stack<int> st;
  
    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[i] > nums[st.top()]) {
            result[st.top()] = nums[i];
            st.pop();
        }
        st.push(i);
    }
    return result;
}
```

### 例2：柱状图最大矩形

- 高度：出栈元素的高度
- 宽度：当前位置 - 栈顶位置 - 1

**寻找当前高度对应的下一个更小的高度**。那么在这个答案对应的下标到栈顶对应的下标这段距离，就是当前高度对应的最大宽度。而当前高度也即是下标取值在这个范围内矩形高度取值的最大值（短板效应），所以这里的高度乘以宽度得到的矩形面积就是在下标取值在这个范围内的最大面积。

```cpp
int largestRectangleArea(vector<int>& heights) {
    stack<int> st;
    heights.push_back(0);  // 哨兵，这是为了保证最后一个元素也能出栈
    int maxArea = 0;
  
    for (int i = 0; i < heights.size(); i++) {
        while (!st.empty() && heights[i] < heights[st.top()]) {
            int h = heights[st.top()];
            st.pop();
            int w = st.empty() ? i : i - st.top() - 1;
            maxArea = max(maxArea, h * w);
        }
        st.push(i);
    }
    return maxArea;
}
```

### 例3：接雨水

```cpp
int trap(vector<int>& height) {
    stack<int> st;
    int water = 0;
  
    for (int i = 0; i < height.size(); i++) {
        while (!st.empty() && height[i] > height[st.top()]) {
            int bottom = st.top();
            st.pop();
            if (st.empty()) break;
          
            int distance = i - st.top() - 1;
            int h = min(height[i], height[st.top()]) - height[bottom];
            water += distance * h;
        }
        st.push(i);
    }
    return water;
}
```

### 例4: 最大二叉树
```cpp
TreeNode* constructMaximumBinaryTree(vector<int>& nums) {
        stack<TreeNode*>st;
        for(auto &i: nums){
            TreeNode* cur = new TreeNode(i);
            while(!st.empty() && st.top()->val<i){
                cur->left=st.top();
                st.pop();
            } 
            if(!st.empty()){
                st.top()->right=cur;
            }
            st.push(cur);
        }
        while(st.size()>1){
            st.pop();
        }
        return st.top();
    }
``` 

## 4. 常用技巧

### 技巧1：哨兵

在数组末尾加 0 或无穷大，确保所有元素都能出栈：

```cpp
heights.push_back(0);
```

### 技巧2：循环数组

遍历两遍模拟循环：

```cpp
for (int i = 0; i < 2 * n; i++) {
    // 使用 nums[i % n]
    if (i < n) st.push(i);  // 只在第一轮入栈
}
```

### 技巧3：存储下标 vs 存储值

- 需要计算距离/面积：存储下标
- 只需要元素值：存储值

## 5. 解题细节

* 递增栈还是递减栈？

* 弹出元素的条件？

* 弹出时如何处理，进而得到答案？

* 是否需要哨兵？
