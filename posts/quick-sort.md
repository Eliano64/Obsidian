---
title: Quick Sort
tags:
  - Sort
categories:
  - Data Structure & Algorithm
date: 2025-08-25 00:00:00
katex: true
---

快速排序是一种平均时间复杂度$O(n\log n)$平均空间复杂度$O(\log n)$的优秀的排序算法。其基本原理是每次维护两个区间里的元素然后不断递归。基本思想与流程是简单明晰的，常见的是左右指针向中间靠拢的方式。但这种方式有很多细节需要注意，例如是左指针先动还是右指针先动、指针移动条件等。稍有不慎，就可能甚至导致死循环。

于是参考《算法导论》给出的更易于理解的做法。

# 1. 伪代码示例

这里是从小到大的顺序排序。

```plaintext
// input: A[lo...hi-1], lo, hi
// A[lo...hi-1]: array to be sorted
// lo, hi: the region to be sorted is [lo, hi)
// output: sorted arr
QuickSort(A[lo...hi-1], lo, hi):
    if lo + 1 >= hi; then
        return // in this case, no element/ only one in the region, no need to sort
    end if
    mi := Partition(A, lo, hi)
    QuickSort(A, lo, mi)
    QuickSort(A, mi + 1, hi)

// output: index where pivot should locate
Partition(A[lo...hi-1], lo, hi):
	pivot := A[hi-1];
	// x \in [lo, i], A[x] < pivot
	i := lo - 1;
	// x \in [i+1, j), A[x] >= pivot
	j := lo 
	while j < hi - 1; do
		if A[j]<pivot; then// then region gets bigger as i increased, and A[j] should be in A[lo..i]
            swap(A[j],A[i+1])
            i = i + 1
		end if
        j = j + 1
	end loop
//i + 1 should be the first index where element >= pivot
	swap(A[i+1],A[hi-1]);
    return i+1;

```

# 2. 解释

在 `Partition`内部每次刚进入循环时，$A$有4个部分：

$$
[lo, i], [i + 1,  j) [j, hi - 2], \{hi - 1\}
$$

其中，第一个区间内的元素$\lt pivot$；第二个$\geq pivot$；第三个未知；第四个是$pivot$的位置。
这四个区间的语义恒定，这就是**循环不变量。**
我们要做的就是逐步探索、逐步扩大第一个和第二个区间内的元素。缩小第三个区间。
如何扩大？$j$遍历数组，第二个区间的上界自然逐步扩大；若发现$A[j]<pivot$则将它与$A[i+1]$交换并让$i=i+1$,这样$A[j]$便加入了第一个区间。
当第二个区间的上界达到$hi -2$时，意味着$hi-1$前面的元素已经整理完毕，应该找到**最小大于等于$pivot$元素的下标x**，这便是$pivot$应该待的位置。而显然，根据各区间的语义，$x = i + 1$。将$A[hi - 1]$与$A[x]$交换，完成最后一步。


# 3. 补充


根据**循环不变量**的思想，也可以有下面左右指针的版本：

```Cpp
inline int _partition(int *A, int lo, int hi) {
	int pivot = A[lo];
	int i = lo;
	int j = hi - 1;
	//lo [lo+1, i] [i+1, j] [j+1,hi-1]
	while(i<j){
    //in my version, there is no stipulation that j must move before i. 
		while(i<j&&A[i+1]<pivot) ++i;
		while(i<j&&A[j]>=pivot) --j;
		if(i<j){
			swap(A[i+1],A[j]);
		}
	}
  // now unknown region is empty, and j is the first index where A[j] >= pivot, 
  //A[lo] = pivot, A[lo+1...i] < pivot
  // so A[i-1] is the last index where A[i-1] < pivot after exchange
	swap(A[i],A[lo]);
	return i;
}

void quickSort(int *A, int lo, int hi) {
    if (lo + 1 >= hi) {
        return;
    }
    int mi = _partition(A, lo, hi);
    quickSort(A, lo, mi);
    quickSort(A, mi+1, hi);
}
```
