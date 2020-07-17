# Python-简单题目
一些基础题目解答的过程，主要是学习。
### 1480. Running Sum of 1d Array
#### 描述
Given an array nums. We define a running sum of an array as runningSum[i] = sum(nums[0]…nums[i]).
Return the running sum of nums.
#### 要求示例
Example 1:Input: nums = [1,2,3,4];Output: [1,3,6,10]
Explanation: Running sum is obtained as follows: [1, 1+2, 1+2+3, 1+2+3+4].

Example 2:Input: nums = [1,1,1,1,1];Output: [1,2,3,4,5]
Explanation: Running sum is obtained as follows: [1, 1+1, 1+1+1, 1+1+1+1, 1+1+1+1+1].

Example 3:Input: nums = [3,1,2,10,1];Output: [3,4,6,16,17]
Constraints:1 <= nums.length <= 1000; -10^6 <= nums[i] <= 10^6
#### 解答
```python
class Solution:
    def runningSum(self, nums: List[int]) -> List[int]:
        x = 0 
        for i in range(0,len(nums)):
            x = x + nums[i]
            nums[i] = x
        return nums
```
刚开始做不熟悉LeetCode的模式，自己创建类和方法去解，可以实现功能，但是提交一直报错。刚开始没有用到切片方式，后来才想到，个人感觉还是上面切片方式简单，代码如下：
```python
class Solution:
    num = 0
    list = []
    def runningSum(self,nums):
        for i in nums:
            self.__class__.num += i
            self.__class__.list.append(self.__class__.num)
        print(self.__class__.list)
    def input_list(self):
        list_input = eval(input())
        self.runningSum(list_input)
solution = Solution()
solution.input_list()
```
### 下一题还在做
