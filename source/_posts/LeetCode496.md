title: LeetCode496 使用栈(stack)简化时间复杂度
date: 2017-02-10 12:56:16
tags:
- Python
- stack
categories:
- Leetcode	
photos:	 
- "https://github.com/aldslvda/blog-images/blob/master/leetcode_logo.png?raw=true"
toc: true
comment: true
---

> LeetCode 496 解题报告

<!-- more -->

## LeetCode 496. Next Greater Element I使用栈(stack)简化时间复杂度

这道题的描述如下：

> **496**. Next Greater Element I  
Description  Submission  Solutions  Add to List  
Total Accepted: 3176  
Total Submissions: 5284  
Difficulty: Easy  
Contributors: love\_FDU\_llp  
You are given two arrays (without duplicates) nums1  and nums2 where nums1’s elements are subset of nums2. Find all the next greater numbers for nums1's elements in the corresponding places of nums2.  

>The Next Greater Number of a number x in nums1 is the first greater number to its right in nums2. If it does not exist, output -1 for this number.

>Example 1:  
Input: nums1 = [4,1,2], nums2 = [1,3,4,2].  
Output: [-1,3,-1]  
Explanation:  
    For number 4 in the first array, you cannot find the next greater number for it in the second array, so output -1.  
    For number 1 in the first array, the next greater number for it in the second array is 3.  
    For number 2 in the first array, there is no next greater number for it in the second array, so output -1.  
Example 2:  
Input: nums1 = [2,4], nums2 = [1,2,3,4].  
Output: [3,-1]  
Explanation:  
    For number 2 in the first array, the next greater number for it in the second array is 3.  
    For number 4 in the first array, there is no next greater number for it in the second array, so output -1.  
Note:  
All elements in nums1 and nums2 are unique.  
The length of both nums1 and nums2 would not exceed 1000.  

这里我们假设nums1的长度是m，nums2的长度是n。
看到题目第一眼，首先想到的是一个O(m*n)的解法：  
**解法1**：   
```python
def nextGreaterElement(self, findNums, nums):
    """
    :type findNums: List[int]
    :type nums: List[int]
    :rtype: List[int]
    """
    res = []
    length = len(nums)
    for i in findNums:
        j, flag = 0, False
        while j < length:
            if i == nums[j] and not flag:
                flag = True
            if flag:
                if j < length-1 and nums[j+1] > i:
                    res.append(nums[j+1])
                    flag = False
                    break
            if j == length - 1:
                res.append(-1)
                flag = False
            j += 1
    return res
```
这个解法的想法比较直接，对nums1里的每个数，都在nums2中找到这个数，然后在接下来的数中找第一个比它大的数放到结果List里，如果没找到就把-1放进去。   
这个虽然也能AC把，但是runtime只能击败2.47%的人.....这样的结果明显有时间复杂度更低的解法。

然后我就去Discuss里面找了....  
果然.....   
被我找到了一个O(m+n)的解法，很巧妙地用到了栈。  
**解法2**：  

```python
def nextGreaterElement(self, findNums, nums):
    """
    :type findNums: List[int]
    :type nums: List[int]
    :rtype: List[int]
    """
    d = {}
    ans = [-1] * len(findNums)
    for i, num in enumerate(findNums):
        d[num] = i
    stack = []
    for num in nums:
        while stack and stack[-1] < num:
            top = stack.pop()
            if top in d:
                ans[d[top]] = num
        stack.append(num)
    return ans
```
做一个简单的解释：

- 我们用一个字典d存储所有元素的index。
- 用一个与nums1等长的数组ans表示结果(并将每个元素初始化为-1)
- 假设我们有一个前面所有元素都递减的数列，最后一个数比前面所有的数都大，比如[5, 4, 3, 2, 1, 6]，那么6就是前面这些数的"Next Greater Element" 。
- 我们用一个栈来放置一个递减的数列。
- 每当即将入栈的一个数比栈顶的数大，我们将栈顶的数弹出，一直到即将入栈的数比栈顶的数小，这个过程中弹出的数的"Next Greater Element"就是这个入栈的数。比如：当前栈是[6,4,3,2],即将入栈的数是5，那么2，3，4会被依次弹出, 并且这三个数的"Next Greater Element"是5
- 我们按照上面描述的规则将nums2中的元素依次入栈，在出栈的元素中找nums1中的元素，如果有，就把ans中对应位置的元素设成即将入栈的那个数
- 这样我们只是遍历了一次nums1，遍历了一次nums2，时间复杂度为O(m+n)

这道题的AC率算很高的了，不过我一直都很难想到简化复杂度的方法，这题用栈解决算是比较巧妙的，姑且做个记录吧。


**PS. 我竟然没到一个星期又更新了**  
**真是勤勉啊（并不**   
**=。=**
