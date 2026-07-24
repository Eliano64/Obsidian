---
title: Counting Josephus and Big Integer
tags:
categories:
  - Data Structure & Algorithm
date: 2026-06-24 00:00:00
katex: true
---

# Counting, Josephus, and Big Integer Notes

## 1. Boyer-Moore 多数投票算法

# 代码

```c\c++
int BoyerMoore(int* array, int arraySize)
{
	int candidate=array[0];
	int count=0;
	for(int i=0;i<arraySize;i++)
	{
		if(count==0)
		{
			candidate=array[i];
		}
		if(array[i]==candidate)
		{
			count++;
		}
		else 
		{
			count--;	
		}
	}
	return candidate;
}
```

# 解释
* 摩尔投票法：若有人得票数超过半数，则他的得票数比其他人加起来都多，即一定满足$~count_{ans}-count_{others}>0~$
* 遍历数组，对于每一个$~array[i]~$，先看$~count~$,若$~count=0~$,则$~array[i]~$便是新的$~candidate~$
* 判定$~if(array[i]==candidate)~$，是则$~count++~$，否则$~count--~$。
* 示例
$$~array={1,1,1,2,3,3,3,3}~$$

| i   | array\[i] | candidate | count |
| --- | --------- | --------- | ----- |
| \   | \         | array\[0] | **0** |
| 0   | 1         | **1**     | 1     |
| 1   | 1         | 1         | 2     |
| 2   | 1         | 1         | 3     |
| 3   | 2         | 1         | 2     |
| 4   | 3         | 1         | 1     |
| 5   | 3         | 1         | **0** |
| 6   | 3         | **3**     | 1     |
| 7   | 3         | 3         | 2     |
 
 	 






如果题目不保证多数元素存在，需要再遍历一次验证 `candidate` 的出现次数是否真的超过一半。

```cpp
int cand = BoyerMoore(array, arraySize);
int freq = 0;
for (int i = 0; i < arraySize; i++)
{
    if (array[i] == cand) freq++;
}
if (freq * 2 > arraySize)
{
    // cand 是多数元素
}
```

## 2. Josephus 问题


candidate={0,1,2,3,...,n-1},围成一个圈。1~m顺序报数，报到m者出列。

求：最后剩下的人的编号。

# 1. 递归解法

## 1.1. 代码

```C\C++
int Josephus(int n,int m)
{
	if(n==1)
	{
		return 0;
	}
	else
	{
		return (Josephus(n-1,m)+m)%n;
	}
}
```

## 1.2. 解释

1. 递归基线：n=1时，survivor=0;
2. 递归条件：n>1时，假设子问题是$~\mathrm{sub\_candidate}=\{0,1,2,3,\dots,n-2\}~$，有$~n-1~$个数;
	然后$~\mathrm{candidate}=\{0,1,2,3,\dots,n-1\}~$，出列第一个数$~m-1~$后，$~\mathrm{candidate}~$也有$~n-1~$个数，即变为$$~\begin{align}&\mathrm{candidate}=\{0,1,\dots,m-2,m,\dots,n-1\}\\=&\{m,m+1,\dots,n-1,0,1,\dots,m-2\}\\=&\{(0+m)\%n,(1+m)\%n,\dots,(n-1-m+m)\%n,(n-m+m)\%n,(n-m+1+m)\%n, \\
&\dots,(n-2+m)\%n\}\\=&(\mathrm{sub\_candidate}+m)\%n\end{align}~$$
	
	既然子问题与原问题存在这样一个递推关系，那么容易证明，它们的解也满足这样的递推关系，所以`Josephus(n,m)=(Josephus(n-1,m)+m)%n`

时间复杂度：$O(n)$

# 2. 通法

将所有出局的人删除，最后剩下的就是答案。

这种解法具有最好的可移植性。例如，将原问题改成，“求最后k个剩下的人的编号”。

## 2.1. 每局出局的人在剩下的人中的编号

$remains$为第$i$局时剩下的人数，$victim$为这一局出局的人的编号。初始化：`victim=(m-1)%candidate.size()`。则下一局出局的人在当前所有人中的编号为`victim=(victim+m-1)%remain`。

## 2.2 代码

### 2.2.1 利用`vector.erase(iterator)`

```C++
int Josephus(int m,vector<int>&candidate)
//candidate[i]=i
{
	int vic=(m-1)%candidate.size();
	while(candidate.size()>1)
	{
		// 移除被淘汰的人
    candidate.erase(candidate.begin() + vic);
        // 下一轮被淘汰的人在更新后的数组中的序号
        vic = (vic + m - 1) % candidate.size();
	}
	return candidate[0];
}
```

这里的`vector.erase(iterator)`的时间复杂度为$O(n_{1})$,$n_{1}$为剩下的人数。

所以时间复杂度为$O(n^{2})$。

### 2.2.2 利用环型链表

```C++
#include <iostream>
using namespace std;

struct node 
{
	int index;
	node * next;
	node(int x):index(x),next(NULL){}
};
int Josephus(int m,node * prev,int num)
//这里用的是prev,即前一个点，这是为了方便节点的删除操作
{
	int vic=(m-1)%num;
	while(num>1)
	{
		for(int i=0;i<vic;i++)
		{
			prev=prev->next;
		}
		//指针移到此轮被淘汰的人的前一个。
		prev->next=prev->next->next;
		// 移除被淘汰的人
        num--;
        vic = (vic + m - 1) % num;
	}
	// 下一轮被淘汰的人在更新后的数组中的序号
	return prev->index;
}
int main()
{
	int n,m;
	cin>>n>>m;
	node * head=NULL;
	node * prev=NULL;
	for(int i=0;i<n;i++)
	{
		node * candidate=new node(i);
		if(head==NULL)
		{
			head=candidate;
		}
		else
		{
			prev->next=candidate;
		}
		prev=candidate;
	}
	prev->next=head;
	cout<<Josephus(m,prev,n);
	delete prev;
	delete head;
}
```

时间复杂度为$O(nm)$。


递归式也可写成迭代，避免递归栈。

```cpp
int JosephusIter(int n, int m)
{
    int ans = 0;
    for (int i = 2; i <= n; i++)
    {
        ans = (ans + m) % i;
    }
    return ans;
}
```

若题目编号从 `1` 开始，则返回 `JosephusIter(n, m) + 1`。

## 3. 大数计算

 由于即使是`unsigned long long int`表示的数亦有极限，所以对于很大的数，用`string`来表示它，并模拟加法，乘法等运算。

这里只介绍加法和乘法。

# 1. 加法

## 1.1 模拟过程

结果为`result`，初始化为空。

当前位的两个数相加，同时加上从低位来的进位。（若当前位为最低位，则进位为0）。求得的和$num=10*a+b$。则$a$为这一位向上一位的进位，$b$为这一位的结果，加在`result`前面。

所有位都计算结束后，如果最高位仍有进位，则在`result`前面加上进位。

为了模拟的方便，可以预先将位数少的数前面补足`"0"`。

返回`result`。

## 1.2 代码

```C++
string add(string big, string small)
{
	string result;  
	//将较大的数赋值给a，较小的数赋值给b
	if (small.size() > big.size())
	{
		swap(big, small);
	}
	int l_len = big.size();
	int s_len = small.size();
	//补足零，方便计算
	for (int i = 0; i < l_len - s_len; i++)
	{
		small = '0' + small;
	}

	int num;        //本位
	int up = 0;     //进位
	for (int i = l_len - 1; i >= 0; i--)
	{
		int num = (big[i] - '0') + (small[i] - '0') + up;
		up = num / 10;
		num %= 10;
		result = (char)(num + '0') + result;
	}
	//模拟加法
	if (up!=0)
	{
		result = (char)(up + '0') + result;
	}
	//若仍有进位，最前面加上进位

	return result;
}
```

# 2. 乘法

## 2.1 模拟过程

结果为`result`，初始化为空。

位数少的数`small`从最后一位开始，逐项与位数多的数`big`相乘。

对于`small`的每一位相乘的过程：

每一位相乘的结果为`part`，初始化为空。

$big$从后往前，依次与$small$的这一位相乘，同时加上从低位来的进位。（若当前位为最低位，则进位为0）。求得的结果$num=10*a+b$。则$a$为这一位向上一位的进位，$b$为这一位的结果，加在`part`前面。

$big$的所有位都计算结束后，如果最高位仍有进位，则在`part`前面加上进位。

`part`根据`small`的这一位数的位置，在后面加上相应个数的`"0"`。

`result=add(result+part)`

这样，将`small`遍历完成后，所有`part`的和`result`即为两数的乘积。

返回`result`。

## 2.1 代码

```C++
string mul(string big, string small)
{
	string result;	
	if (small.size() > big.size())
	{
		swap(big, small);
	}
	int l_len = big.size();
	int s_len = small.size();
	int num;        //本位
	for (int i = s_len - 1; i >= 0; i--)
	{
		int up = 0;		//进位
		string part="";
		for (int j = l_len - 1; j >= 0; j--)
		{
			num = (big[j] - '0') * (small[i] - '0') + up;
			up = num / 10;
			num %= 10;
			part = (char)(num + '0') + part;
		}
		if (up!=0)
		{
			part = (char)(up + '0') + part;
		}
		//若仍有进位，加上进位
		for (int k = 0; k < s_len - 1 - i; k++)
		{
			part += '0';
		}
		result = add(result, part);
		//求和
	}
	//乘法过程：小的数从最后一位开始，逐项与大数相乘，最后乘10的k次方，求和。
	return result;
}
```

# 3. 推广

* 该方法当然可以推广到算小数，即高精度计算。只需要在最后另外考虑小数点的位置即可。
* 也可以推广到n进制的大数计算。而且加法的话改动还不大。


# 4. 频次统计：最大公共元素数量


问题特点：它和“最长公共子序列”看起来相似，但**允许顺序调换**，因此不需要维护序列顺序，也不需要 DP。

思路：用两个 `unordered_map` 记录两个数组中每个元素出现的次数。遍历其中一个频次表，如果另一个表中也有该元素，则答案加上两个出现次数的较小值。

```cpp
#include <bits/stdc++.h>
using namespace std;

int findCommon(vector<int>& S, vector<int>& T)
{
    unordered_map<int, int> freq1, freq2;

    for (const auto& elem : S)
    {
        freq1[elem]++;
    }

    for (const auto& elem : T)
    {
        freq2[elem]++;
    }

    int ans = 0;
    for (const auto& kv : freq1)
    {
        if (freq2.find(kv.first) != freq2.end())
        {
            ans += min(kv.second, freq2[kv.first]);
        }
    }

    return ans;
}
```

| 问题       | 是否要求顺序   | 方法         |
| -------- | -------- | ---------- |
| 最长公共子序列  | 要求保持相对顺序 | 动态规划       |
| 最大公共元素数量 | 不要求顺序    | 哈希表 / 频次统计 |
