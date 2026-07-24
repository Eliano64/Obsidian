---
title: binary search
tags:
  - Search
categories:
  - Data Structure & Algorithm
date: 2025-07-21 00:00:00
katex: true
---

# 1. 总结

二分查找的原理非常简单，但是一些细节例如是 `l<r`还是 `l<=r`、更新 `r`时是 `r=mid`还是 `r=mid-1`（`l`同理）等地方却有些让人头疼，实际写来如果不注意就可能会造成死循环。

于是总结一种模板：

**定义域为\[lo, hi)的单增的f(x), 找出最小的ans, 使得f(ans)>0成立。**

单减同理，甚至可以进行预处理先转化为单增的情况。

伪代码如下：

```plaintext
algorithm binary-search(lo,hi)
	while the search area has elments do:
		mid <- lo + (hi-lo)/2;
		if f(mid) satisfied: // the answer may occur here
            		ans := mid;
			hi <- mid; 
            // the search area could have no elments when in the next loop, so return ans; 
            // but mid>ans if next loop continues,  
            // for this is a right-open area, hi <- mid;
		else :
			lo <- mid + 1; 
            // mid is not the answer, and mid < answer;
            // for this is a left-close area, lo <- mid + 1;
	end while
	return ans;

```

# 2. 例

1. 已知一个有序数组，查找一个元素的位置并返回。若该元素不存在，则返回元素应插入的位置。

**定义域为\[0, nums.size())，找出最小的ans, nums\[ans\] >= target成立**

```C
int searchInsert(int[] nums, int numSize, int target)
{
        int left=0,right=numSize,ans=numSize-1; // f(ans)= nums[ans] 的定义域： ans \in [left, right)
        while(left<right){
            int mid=(right-left)/2 + left;//防止直接right+left造成溢出
            if(nums[mid]>=target){//因为我们要找的就是nums[ans]>=target
                ans=mid; //所以mid可能为答案。
                right=mid; //也可能mid不是答案，更新开的上界为mid
            }
            else{
                left=mid+1; // 否则mid绝对不是答案。更新闭的下界为mid+1。
            }
        }
        return ans;
 }
```

2. 给你一个非负整数 `x` ，计算并返回 `x` 的 **算术平方根** 。
   由于返回类型是整数，结果只保留 **整数部分** ，小数部分将被 **舍去 。**

**定义域为$[0, +\infty)$, 找出最大的$\text{ans}$, $\text{ans}^2 \leqslant x$**

```C
int mySqrt(int x) {
       long long l = 0, r = x+1, ans; // ans \in [0, +\infty), 但这里对r这样赋值只是为了让ans快速收敛。为什么x+1: x=0,1时 ans = x < x+1。  
        while (l < r) {
            long long  mid = (r - l)/2 + l;
            if (mid * mid <= x) {//因为要找的就是ans*ans<=x
                ans = mid; //所以mid可能为答案。
                l = mid + 1; //也可能mid不是答案，更新闭的下界为mid+1
            } else {
                r = mid; // 否则mid绝对不是答案。更新开的上界为mid。
            }
        }
        return ans; 
    }
```

也有一些**更抽象**的二分查找：

[3350.检测相邻递增子数组II](https://leetcode.cn/problems/adjacent-increasing-subarrays-detection-ii/description/)

令f(x)：`k=x`时数组是否存在两个相邻且长度为 `k` 的严格递增子数组,存在则`f(x)=1`否则`f(x)=0`。

则`f(x)`单减。**满足二分查找的使用**

于是原问题即为

**定义域为$[2, \lceil nums.length/2 \rceil)$, 找出最大的$\text{ans}$, $f(\text{ans})>=1$**

解法示例：

```go
func f(x int, lengths []int) bool {
    count := 0
    for _, l := range lengths {
        if l >= 2*x { 
            return true
        }
        if l >= x {
            count++
            if count >= 2 {
                return true
            }
        }else{
            count = 0
        }
    }
    return false
}

func maxIncreasingSubarrays(nums []int) int {
    n := len(nums)
    lengths := []int{}
    cur := 1
    for i := 1; i < n; i++ {
        if nums[i] > nums[i-1] {
            cur++
        } else {
            lengths = append(lengths, cur)
            cur = 1
        }
    }
    lengths = append(lengths, cur)

    lo := 1; hi := len(nums)/2+1
    var ans int
    for lo<hi{
        mi := lo + (hi-lo)/2
        if f(mi,lengths){
            ans = mi
            lo = mi + 1
        }else{
            hi=mi
        }
    }
    return ans
}
```

# 3. 何时使用二分查找？

- 寻找一个*值*
- 题目条件给出数组，以及建立在数组上的一个**单调**映射f。
- 这个*值*使得f*恰好*满足某个条件。
