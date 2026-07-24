---
title: reverse linked list
tags:
  - LinearList
categories:
  - Data Structure & Algorithm
date: 2025-09-26 00:00:00
katex: true
---

# 1. 操作

以[这道题](https://leetcode.cn/problems/reverse-linked-list-ii/description/)为实例。

对链表的某一部分进行翻转时，需要记录以下这几个节点：
- `p0`: 发生翻转区域的前置节点，这个是为了区域翻转后的连接（**循环结束，`p0.next`为翻转后区间的最后一个元素**）。为了统一处理（**因为原链表有的翻转区域没有前置节点**），先构造`dummy=ListNode{next: head}`。这样的话，翻转后的链表的头节点即为`dummy.next`。p0可能为dummy。
- `cur`: 当前遍历到的节点，**循环结束，`cur`为区域后的第一个元素**
- `pre`: 当前遍历到的节点的原链表中的上一个节点，在翻转后就是`next`。**在最初翻转前为`nullptr`,循环结束，`pre`为翻转后的区域的第一个元素**
- `nxt`: 当前遍历到的节点的原链表中的下一个节点，下一个`cur`，在翻转后它的`next`就是`pre`。

操作如下：

```text
cur := p0.next
pre := nullptr
for cur in area; do
    nxt := cur.next
    cur.next = pre
    pre = cur
    cur = nxt
end loop
// one loop only change one "next"
// end loop, cur is next node for this area at the time
p0.next.next=cur // set next pointer of the reversed area's last node  
p0.next=pre // previous node's next pointer points to the first node 
```

# 2. 例题

[25.k个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/description/)

这个题的有一个关键：找到每一个翻转区间的`p0`。**它实际就是上一个区间未翻转时的第一个节点。**

go示例解法

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func reverseKGroup(head *ListNode, k int) *ListNode {
    cnt := 0
    for p:=head;p!=nil;p=p.Next{
        cnt++
    }
    dummy := &ListNode{0,head}
    p0 := dummy
    for ; cnt>=k ; cnt-=k {
        cur := p0.Next
        var pre *ListNode
        nextp := p0.Next
        for i:=0;i<k;i++{
            nxt := cur.Next
            cur.Next = pre
            pre = cur
            cur = nxt
        }
        p0.Next.Next=cur
        p0.Next = pre
        p0=nextp
    } 
    return dummy.Next
}
```
