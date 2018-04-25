title: 利用Python位运算简化时间/空间复杂度
date: 2017-02-24 16:33:08
tags:
- Python
- Bitwise Operation
categories:
- Leetcode	
thumbnail:	"https://github.com/aldslvda/blog-images/blob/master/leetcode_logo.png?raw=true"
toc: true
comment: true
---
![title](https://github.com/aldslvda/blog-images/blob/master/leetcode_logo.png?raw=true)
最近两天碰到两个用位运算解决的题目，恰好一个利用位运算简化了时间复杂度，另一个简化了空间复杂度，所以做个记录加深一下印象。
第一个题目是LeetCode 421. Maximum XOR of Two Numbers in an Array:  
**注：这题中提到的异或均是按位异或。
> Given a non-empty array of numbers, a0, a1, a2, … , an-1, where 0 ≤ ai < 231.

> Find the maximum result of ai XOR aj, where 0 ≤ i, j < n.

> Could you do this in O(n) runtime?

> Example:

> Input: [3, 10, 5, 25, 2, 8]

> Output: 28

> Explanation: The maximum result is 5 ^ 25 = 28.

这题的描述中明确指出了**do this in O(n) runtime** ，如果要两两异或比较大小的话很明显需要O(n^2)的时间复杂度。

那怎么样才能用位运算解决这个问题呢？

如果我们遇到的场景是这样：

> 假定数组中所有的元素都可以用8位2进制数来表示，如果我们已经知道所有数前7位的最大异或值maxor，怎样求8位数的最大异或值？

很容易想到8位数的最大异或值只可能是前7位后面跟上0或1， 所以我们先假设后面能跟上1，即**假设的**最大异或值为
 
> maxornew =（maxor << 1）+ 1;

我们要找到这个最大异或的值会不会出现，一种简单的办法是2个循环遍历数组，然后两两算出异或的值，看是否能找到一个结果与maxornew相同。但是这样会使时间复杂度变成O(n^2)，不满足题目的要求。
但是按位异或有一个比较特殊的性质：

> 若 x ^ y = z ,那么 x ^ z = y , y ^ z = x

这样我们只用拿出数组中每一个八位二进制数， 与maxornew进行异或运算，再判断异或得到的结果是否在数组中，如果不在的话，最大的异或值就是(maxor<<1) ,这样时间复杂度就被简化成为了O(n)

上面的说完了，这个题目中描述的问题也就迎刃而解了。题目中提到的numbers，由于在给出的函数声明中传入参数为List[int], LeetCode中这样的数一般认为是 32位的int。

对numbers中每一个数取前n位(1<=n<=32)计算最大异或的值，并且按照上面计算第八位的方法计算下一位，就可以得到整个数组的最大异或值。

```python
def findMaximumXOR(self, nums):
    answer = 0
    for i in range(32)[::-1]:
        answer <<= 1
        prefixes = {num >> i for num in nums}
        answer += any(answer^1 ^ p in prefixes for p in prefixes)
        #print bin(answer)[2:]
    return answer
```
这题利用位运算将原本需要O(n^2)的时间复杂度简化成了O(n)

============================================================================

第二个题目是简化空间复杂度，LeetCode 137. Single Number II：

> Given an array of integers, every element appears three times except for one, which appears exactly once. Find that single one.

> Note:
> Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory? 

就是说一个数组中，除了其中一个数只出现一次，其他都出现了三次，找出这个只出现了一次的数。
首先我想到的是一个时间复杂度为O(n),使用的额外空间也是O(n)的解法：

```python
def singleNumber(self, nums):
    """
    :type nums: List[int]
    :rtype: int
    """
    numdict = {}
    for i in nums:
        if i in numdict:
            numdict[i]+=1
        else:
            numdict[i]=1
    for i in nums:
        if numdict[i]==1:
            return i
```
简单来说就是遍历数组的同时记录每个数出现的次数，第二次遍历的时候找出出现了一次的数。

如果要将空间复杂度简化成O(1), 使用位运算是一个很好的选择：
1. 将所有数转换成32位的2进制表示
2. 把每一位的值加起来对3取余
3. 转换成十进制表示
4. Python将负整数转换成二进制数的时候，直接是'-'加上这个数的绝对值的二进制表示，所以负号单独计算。
代码如下：

```python
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        sumlist = [0]*32
        summary, negcnt = 0, 0
        for i in nums:
            negcnt += i < 0
            i = abs(i)
            bitwise = format(i, '032b')
            for j in xrange(32):
                sumlist[j] += int(bitwise[j])
        for i in xrange(32):
            summary += 2**(31-i)*(sumlist[i]%3)
        return summary*[1,-1][negcnt%3]
```

和上面的方法比起来，额外空间简化成了常数， 即O(1)。

不过话说回来，这是一种用时间换空间的做法，上面的做法时间是2n， 这个是32n，虽然复杂度都是O(n),但是实际运行时间差距还是比较大的。


（ · x · ）~