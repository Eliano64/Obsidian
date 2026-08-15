---
title: monotonic stack
tags:
  - Stack
categories:
  - Data Structure & Algorithm
date: 2025-07-21 00:00:00
katex: true
---

> 他向远方望去，无法看到高山背后的矮山，只看到一座座更高的山峰。————by [灵神](https://leetcode.cn/discuss/post/3579480/ti-dan-dan-diao-zhan-ju-xing-xi-lie-zi-d-u4hk/)

# 模型建立

> [!example] 单调栈处理的问题
> 给定一个序列，对每个元素寻找能够向一侧延伸到的边界。例如：为每个 `a[i]` 寻找右侧第一个大于它的元素。

# 单调栈

从左向右扫描数组，后进入的待解决元素先得到答案，符合[[Stack-Definition-And-Operations#栈的语义|栈的语义]]。

以“寻找右侧第一个更大元素”为例。扫描到 `a[i]` 时，若它大于栈顶元素，那么栈顶元素找到了栈顶元素右侧第一个更大的元素，找到了栈顶元素对应的问题的答案（此前扫描过的中间元素都没有使栈顶出栈，因此它们都不满足条件），记录答案并弹出栈顶；若 `a[i]` 仍大于新的栈顶，就继续处理。最后把 `i` 入栈，等待后面的元素回答它的问题。

弹出所有小于 `a[i]` 的元素后，栈中对应的值从栈底到栈顶保持单调不增，所以称为**单调栈**。


```txt
algorithm monotonic-stack(A, ans)
    stack S ← ∅
    ans[n]  ← ∅
    for i ← 0 to n-1 do:
        while S ≠ ∅ and condition(A[i], A[S.top()]) do: // A[i] 已成为栈顶元素的答案
            j ← S.pop()
            recordAnswer(j, i)
        end while
        S.push(i) // 当前元素等待后续元素给出答案
    end for
    return ans
end algorithm
```

### 复杂度

空间复杂度$O(n)$: 需要 `ans[n]` 和栈。
时间复杂度$O(n)$: 每个元素最多入栈一次，出栈一次，总操作数 ≤ 2n。

# 典型例题

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
