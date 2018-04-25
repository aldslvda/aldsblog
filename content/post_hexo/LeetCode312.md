title: LeetCode312 分治法和动态规划
date: 2017-03-09 15:24:09
tags:
- Python
- Divide and Conquer
- Dynamic Programming
categories:
- Leetcode	
thumbnail:	"https://github.com/aldslvda/blog-images/blob/master/leetcode_logo.png?raw=true"
toc: true
comment: true
---
![title](https://github.com/aldslvda/blog-images/blob/master/leetcode_logo.png?raw=true)
最近开始渐渐忙了起来，加上按Acceptance做题，后面题目也开始有点复杂了，所以只能勉强维持一天做一道题。今天下午差不多把这周任务完成了，所以稍微划下水，写一下这两天碰到的一个比较复杂的题目（对我来说=。=

题目如下：

> **312**. Burst Balloons   
Total Accepted: 23001   
Total Submissions: 54845   
Difficulty: Hard   
Contributors: Admin   
Given n balloons, indexed from 0 to n-1. Each balloon is painted with a number on it represented by array nums. You are asked to burst all the balloons. If the you burst balloon i you will get nums[left] * nums[i] * nums[right] coins. Here left and right are adjacent indices of i. After the burst, the left and right then becomes adjacent.

> Find the maximum coins you can collect by bursting the balloons wisely.

> Note: 
> (1) You may imagine nums[-1] = nums[n] = 1. They are not real therefore you can not burst them.
> (2) 0 ≤ n ≤ 500, 0 ≤ nums[i] ≤ 100

> Example:

> Given [3, 1, 5, 8]

> Return 167

>    nums = [3,1,5,8] --> [3,5,8] -->   [3,8]   -->  [8]  --> []
   coins =  3 * 1 * 5      +  3 * 5 * 8    +  1 * 3 * 8      + 1 * 8 * 1   = 167
   

1. 看到这个题目，最简单的想法是用回溯法，使用回溯法一共需要n步，第i步需要尝试选出的数有n-i个,这样我们得到的时间复杂度是O(n!)，非常大，所以这个方法没有尝试的必要。
我们需要思考的是这个方法中有没有冗余的步骤可以简化。

2. 我们可以注意到，每次选择爆破的气球，与上一步选择爆破的气球是没有关系的，这满足了动态规划“无后效性”的特点。   
	这样我们可以用从下到上的动态规划来解决这个问题，即计算小长度数组的最大积分值，通过小长度的数组的累加来计算整个数组的最大积分值。这样如果计算长度为k的子数组，需要需要找到的不同情况是C(k, n),需要从长度为1找到长度为n，这样最坏的情况也是O(n!)（。。。。。）   
虽然说没好多少，但是至少稍微简化了一点时间复杂度。。。。

3. 在用上面的这个思路思考问题的时候，又可以注意到，子问题与父问题的解决方式非常类似，可以考虑用**分治法**递归的解决这个问题。   
	说到分治，我们最直观的想到的使用分治的方法是:当一个气球爆裂后，它两边的气球串分别形成爆裂前气球串的子问题。但是分治法要求的是**相互独立的子问题**，而照上面这种方法，由于爆破后，左边气球串的最后一个气球和右边气球串的最左边一个气球相邻了，这样导致的结果是分离出的两个子问题相互影响了，所以这种分治是不可行的。
	如果我们逆向思维一下，找到**最后**爆破的气球，再把它的两边分成两个子问题来解决，可不可行呢？由于最后一个爆破的气球，爆破时长度为1，根据题目定义左右都用1来填充，这样由于额外填充的1，使得这个气球并不会影响子问题的划分，这样的分治是可行的。

4. 最后我们选择了由下而上的**动态规划**和**分治法**解决题目所提到的问题。代码如下：

```python
class Solution(object):
    def maxCoins(self, nums):
    	#增加气球串的边界
        coins = [1] + [num for num in nums if num] + [1]
        n = len(coins)
        dp = [[0] * n for _ in xrange(n)]

        for l in xrange(2, n):
            #计算每个长度为l的子序列
            for left in xrange(0, n-l):
                right = left + l
                for j in xrange(left+1, right):
                #由于是自下而上的动态规划，所以免去了分治法中使用递归解决子问题这一步
                #coins[left] * coins[j] * coins[right] 这个表达式是指爆破最后一个气球产生的运算
                    dp[left][right] = max(dp[left][right], dp[left][j] + \
                            dp[j][right] + coins[left] * coins[j] * coins[right])
        return dp[0][n-1]
```

这样一个比较复杂的问题就迎刃而解了，这样做的时间复杂度是O(n^3)。   
总结起来就是：非常暴力的方法 -->简化冗余--> 不那么暴力的方法 -->简化冗余--> 比较简单的方法 -->简化冗余--> 很好的方法   
其实这个问题我想了很久都没想出来，最后在discuss里看到这个比较好的解决方式，这个问题的思考过程也是非常值得借鉴的，所以在这里记录一下,也尽我所能讲的比较明白一些。


(溜了