# Python-简单题目
一些基础题目解答的过程，主要是学习。
### 1480. Running Sum of 1d Array
#### 描述
Given an array nums. We define a running sum of an array as runningSum[i] = sum(nums[0]…nums[i]).
Return the running sum of nums.
#### 要求示例
- Example 1:Input: nums = [1,2,3,4];Output: [1,3,6,10]
Explanation: Running sum is obtained as follows: [1, 1+2, 1+2+3, 1+2+3+4].
- Example 2:Input: nums = [1,1,1,1,1];Output: [1,2,3,4,5]
Explanation: Running sum is obtained as follows: [1, 1+1, 1+1+1, 1+1+1+1, 1+1+1+1+1].
- Example 3:Input: nums = [3,1,2,10,1];Output: [3,4,6,16,17]
- Constraints:1 <= nums.length <= 1000; -10^6 <= nums[i] <= 10^6

#### 解答
提交代码如下：
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
### 1512. Number of Good Pairs
#### 描述
Given an array of integers nums.
A pair (i,j) is called good if nums[i] == nums[j] and i < j.
Return the number of good pairs.
#### 示例和要求
- Example 1:Input: nums = [1,2,3,1,1,3];Output: 4;Explanation: There are 4 good pairs (0,3), (0,4), (3,4), (2,5) 0-indexed.
- Example 2:Input: nums = [1,1,1,1];Output: 6;Explanation: Each pair in the array are good.
- Example 3:Input: nums = [1,2,3];Output: 0
- Constraints:1 <= nums.length <= 100;1 <= nums[i] <= 100

#### 解答
提交代码如下：
```python
class Solution:
    def numIdenticalPairs(self, nums: List[int]) -> int:
        pair = ((0,3),(0,4),(3,4),(2,5))
        count = 0
        for (i,j) in pair:
            i = (i,j)[0]
            j = (i,j)[1]
            if nums[i] == nums[j] and i < j:
                count += 1
        return count
```
### 下一题解答中
