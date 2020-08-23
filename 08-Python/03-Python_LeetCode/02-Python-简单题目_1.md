# Python-简单题目
一些Leetcode简单题目解答的过程。题目的描述和示例有点多，占位置，详细可以在题目原链接上查看。
### 1486. XOR Operation in an Array  
题目链接：[https://leetcode.com/problems/xor-operation-in-an-array/](https://leetcode.com/problems/xor-operation-in-an-array/)
#### 说明
Given an integer n and an integer start.       
Define an array nums where nums[i] = start + 2*i (0-indexed) and n == nums.length.         
Return the bitwise XOR of all elements of nums.          
#### 解答
提交代码如下：
```python
class Solution:
    def xorOperation(self, n: int, start: int) -> int:
        from functools import reduce
        nums = []
        for i in range(0,n):
            num = start + 2 * i
            nums.append(num)
        vaule = reduce(lambda x,y:x^y,nums)
        return vaule
```
### 下一题解答中
