# Python-简单题目
一些基础题目解答的过程。
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
        count = 0
        for i in range(0,len(nums)):
            for j in range(0,len(nums)):
                if nums[i] == nums[j] and i < j:
                    count += 1
        return count
```
### 1365.How Many Numbers Are Smaller Than the Current Number

#### 描述
Given the array nums, for each nums[i] find out how many numbers in the array are smaller than it. That is, for each nums[i] you have to count the number of valid j's such that j != i and nums[j] < nums[i].
（总感觉后面这句描述有问题，还是理解有问题。）
Return the answer in an array.

#### 解答
```python
class Solution:
    def smallerNumbersThanCurrent(self, nums: List[int]) -> List[int]:
        out_list = []
        for i in range(0,len(nums)):
            count = 0
            for j in nums:
                if  j != nums[i] and j < nums[i]:
                    count += 1
            out_list.append(count)
        return out_list
```
### 1470.Shuffle the Array
#### 描述
Given the array nums consisting of 2n elements in the form [x1,x2,...,xn,y1,y2,...,yn].     
Return the array in the form [x1,y1,x2,y2,...,xn,yn].   
Example 2:Input: nums = [1,2,3,4,4,3,2,1], n = 4      
Output: [1,4,2,3,3,2,4,1]     
#### 解答
提交代码如下：
```python
class Solution:
    def shuffle(self, nums: List[int], n: int) -> List[int]: 
        out_list = []
        for i in range(0,n):
            j = i + n
            x = nums[i]
            out_list.append(x)
            y = nums[j]
            out_list.append(y)
        return out_list
```
### 1431.Kids With the Greatest Number of Candies
#### 描述
Given the array candies and the integer extraCandies, where candies[i] represents the number of candies that the ith kid has.         
For each kid check if there is a way to distribute extraCandies among the kids such that he or she can have the greatest number of candies among them. Notice that multiple kids can have the greatest number of candies.   
#### 解答
提交代码如下：
```python
class Solution:
    def kidsWithCandies(self, candies: List[int], extraCandies: int) -> List[bool]: 
        out_list = []
        for i in range(0,len(candies)):
            out_list.append((candies[i] + extraCandies >= max(candies)))
        return out_list    
```
### 1672.Richest Customer Wealth
#### 描述
&#8195;You are given an `m x n` integer grid accounts where `accounts[i][j]` is the amount of money the `i​​​​​​​​​​​th`​​​​ customer has in the `j​​​​​​​​​​​th`​​​​ bank. Return the wealth that the richest customer has.
&#8195;A customer's wealth is the amount of money they have in all their bank accounts. The richest customer is the customer that has the maximum wealth.
#### 解答
提交代码如下：
```python
class Solution:
    def maximumWealth(self, accounts: List[List[int]]) -> int:
        wealth_list = []
        for i in range(0,len(accounts)):
            wealth_sum = sum(accounts[i])
            wealth_list.append(wealth_sum)
        return max(wealth_list)
```
### 1108.Defanging an IP Address
#### 描述
&#8195;Given a valid (IPv4) IP address, return a defanged version of that IP address.A defanged IP address replaces every period "." with "[.]".
#### 解答
提交代码如下：
```python
class Solution:
    def defangIPaddr(self, address: str) -> str:
        import re
        new_add = re.sub('\.','[.]',address,0)
        return new_add
```
### 771.Jewels and Stones
#### 描述
&#8195;You're given strings`J`representing the types of stones that are jewels,and `S`representing the stones you have. Each character in S is a type of stone you have. You want to know how many of the stones you have are also jewels.      
&#8195;The letters in`J`are guaranteed distinct,and all characters in`J`and`S`are letters. Letters are case sensitive, so"a"is considered a different type of stone from "A".
#### 解答
提交代码如下：
```python
class Solution:
    def numJewelsInStones(self, J: str, S: str) -> int:
        count = 0 
        for i in J:
            for j in S:
                if i == j:
                    count +=1
        return count
```
### 1662.Check If Two String Arrays are Equivalent
#### 描述
&#8195;Given two string arrays`word1`and`word2,return true if the two arrays represent the same string,and false otherwise.      
&#8195;A string is represented by an array if the array elements concatenated in order forms the string.
#### 解答
提交代码如下：
```python
class Solution:
    def arrayStringsAreEqual(self, word1: List[str], word2: List[str]) -> bool:
        i = ''.join(word1)
        j = ''.join(word2)
        if i ==j:
            return True
        else:
            return False
```
### 1603.Design Parking System
#### 描述
&#8195;Design a parking system for a parking lot.The parking lot has three kinds of parking spaces:`big`,`medium`,and`small`,with a fixed number of slots for each size.          
Implement the ParkingSystem class:
- ParkingSystem(int big, int medium, int small) Initializes object of the ParkingSystem class. The number of slots for each parking space are given as part of the constructor.
- bool addCar(int carType) Checks whether there is a parking space of carType for the car that wants to get into the parking lot. carType can be of three kinds: big, medium, or small, which are represented by 1, 2, and 3 respectively. A car can only park in a parking space of its carType. If there is no space available, return false, else park the car in that size space and return true.
#### 解答
提交代码如下：
```python
class ParkingSystem:
    def __init__(self, big: int, medium: int, small: int):
        self.big = big
        self.medium = medium
        self.small = small
    def addCar(self, carType: int) -> bool:
        if carType == 1:
            if self.big > 0:
                self.big -= 1
                return True
        if carType == 2:
            if self.medium > 0:
                self.medium -= 1
                return True
        if carType == 3:
            if self.small > 0:
                self.small -= 1
                return True    
# Your ParkingSystem object will be instantiated and called as such:
# obj = ParkingSystem(big, medium, small)
# param_1 = obj.addCar(carType)
```
### 1313.Decompress Run-Length Encoded List
#### 描述
&#8195;We are given a list`nums`of integers representing a list compressed with run-length encoding.Consider each adjacent pair of elements`[freq, val] = [nums[2*i], nums[2*i+1]]` (with i >= 0).For each such pair,there are`freq`elements with value`val`concatenated in a sublist.Concatenate all the sublists from left to right to generate the decompressed list.   
Return the decompressed list.
#### 解答
提交代码如下：
```python
class Solution:
    def decompressRLElist(self, nums: List[int]) -> List[int]:
        result_list = []
        for i in range(0,len(nums)//2):
            [freq, val] = [nums[2*i], nums[2*i+1]]
            array = freq * [val]
            result_list = result_list + array
        return result_list
```
### 下一题解答中
