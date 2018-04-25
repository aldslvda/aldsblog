title: LeetCode382 蓄水池抽样算法
date: 2017-02-16 12:06:23
tags:
- Python
- 蓄水池抽样
categories:
- Leetcode	
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/leetcode_logo.png?raw=true"
toc: true
comment: true
---

## LeetCode382 蓄水池抽样算法
首先说一下**蓄水池抽样算法**：

- 问题场景

> 给你一个长度为N的链表。N很大，但你不知道N有多大。你的任务是从这N个元素中随机取出k个元素。你只能遍历这个链表一次。你的算法必须保证取出的元素恰好有k个，且它们是完全随机的（出现概率均等）。

- 问题解决  
**蓄水池抽样算法**

> 该算法是针对从一个序列中随机抽取不重复的k个数，保证每个数被抽取到的概率为k/n这个问题而构建的。做法是： -
首先构建一个可放k个元素的蓄水池，将序列的前k个元素放入蓄水池中。
然后从第k+1个元素开始，以k/n的概率来决定该元素是否被替换到池子中。 当遍历完所有元素之后，就可以得到随机挑选出的k个元素。复杂度为O(n).

伪代码如下：

```
Init : a reservoir with the size： k
for  i= k+1 to N
    M=random(1, i);
    if( M < k)
        SWAP the Mth value and ith value
end for
```

- 证明每个数被取到的概率为k/n:

1. 对于第i个数(i < k), 在前k步被选中的概率是1, 从第k+1步开始, i不被选中的概率为k/k+1, 那么读到第n个数时:
> 第i个数(i < k)被选中的概率 = 被选中的概率 * 以后每一步都不被换走的概率  

	即:
> 1 * k/(k+1) * (k+1)/(k+2) … (n-1)/n = k/n

2. 对于第j个数(j >= k): 
> 被选中的概率 = 在他出现时被选中的概率 * 在他出现以后不被换走的概率

	即: 
> k/j * j/(j+1) ... (n-1)/n = k/n

	综上得证。
  
- Leetcode 382 解题报告：

> **382**. Linked List Random Node  
Total Accepted: 20928  
Total Submissions: 45158  
Difficulty: Medium  
Contributors: Admin  
Given a singly linked list, return a random node's value from the linked list. Each node must have the same probability of being chosen.

> Follow up:
What if the linked list is extremely large and its length is unknown to you? Could you solve this efficiently without using extra space?

**Example:**

```
// Init a singly linked list [1,2,3].
ListNode head = new ListNode(1);
head.next = new ListNode(2);
head.next.next = new ListNode(3);
Solution solution = new Solution(head);

// getRandom() should return either 1, 2, or 3 randomly. Each element should have equal probability of returning.
solution.getRandom();
```

这个题就是上面所说的蓄水池长度为1的情况,Python代码如下：

```python
import random
class Solution(object):

    def __init__(self, head):
        """
        @param head The linked list's head.
        Note that the head is guaranteed to be not null, so it contains at least one node.
        :type head: ListNode
        """
        self.head = head

    def getRandom(self):
        """
        Returns a random node's value.
        :rtype: int
        """
        i = 1
        head = self.head
        while head != None:
            M = random.randrange(0, i)
            if M == 0:
                k = head.val
            head = head.next
            i += 1
        return k
```
由于题目中提到
> What if the linked list is extremely large

而对于每个元素都要计算一次随机数, 非常耗时，所以有一个简化的版本，每100个数计算一次随机数, 代码如下：

```python
class Solution(object):

    def __init__(self, head):
        self.head = head

    def getRandom(self):
        node = self.head
        before = 0
        buffer = [None] * 100
        while node:
            now = 0
            while node and now < 100:
                buffer[now] = node
                node = node.next
                now += 1
            r = random.randrange(now + before)
            if r < now:
                pick = buffer[r]
            before += now
        return pick.val
```
已经做到AC率 50% 以下了, 感觉题目虽然都不是很难，但是已经开始涉及我的知识盲区了orz, 所以接下来碰到一些没学过的东西也会记录下来=。=
 
