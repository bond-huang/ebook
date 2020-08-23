# Python-中等难度
一些Leetcode中等难度题目解答的过程。题目的描述和示例有点多，占位置，详细可以在题目原链接上查看。
### 1476. Subrectangle Queries   
题目链接：[https://leetcode.com/problems/subrectangle-queries/](https://leetcode.com/problems/subrectangle-queries/)
#### 说明
Implement the class SubrectangleQueries which receives a rows x cols rectangle as a matrix of integers in the constructor and supports two methods: 
1. updateSubrectangle(int row1, int col1, int row2, int col2, int newValue),Updates all values with newValue in the subrectangle whose upper left coordinate is (row1,col1) and bottom right coordinate is (row2,col2). 
2. getValue(int row, int col),Returns the current value of the coordinate (row,col) from the rectangle. 
#### 解答
第一次测试通过后提交代码如下：
```python
class SubrectangleQueries:
    def __init__(self, rectangle: List[List[int]]):
        self.rectangle = rectangle
    def updateSubrectangle(self, row1: int, col1: int, row2: int, col2: int, newValue: int) -> None:
        for i in range(row1,row2+1):   
            if i == row1 and i == row2:
                for j in range(col1,col2+1):
                     self.rectangle[i][j] = newValue
            elif i == row1 and i != row2:
                for k in range(col1,len(self.rectangle[0])):
                    self.rectangle[row1][k] = newValue
            elif i != row1 and i == row2:
                for l in range(0,col2+1):
                     self.rectangle[row2][l] = newValue
            else:
                for m in range(0,len(self.rectangle[0])):
                     self.rectangle[i][m] = newValue
        return  self.rectangle 
    def getValue(self, row: int, col: int) -> int:
        value = self.rectangle[row][col]
        return value       
```
&#8195;&#8195;测试运行通过，但是提交后报错，也就是不满足题目要求，把题目想复杂了，并且没有认真审题，错误理解是从(row1,col1)到(row2,col2)中间所有的值都要去修改，也没注意有下面要求：
```
0 <= row1 <= row2 < rows
0 <= col1 <= col2 < cols
```
修改后提交代码如下：
```python
class SubrectangleQueries:
    def __init__(self, rectangle: List[List[int]]):
        self.rectangle = rectangle
    def updateSubrectangle(self, row1: int, col1: int, row2: int, col2: int, newValue: int) -> None:
        for i in range(row1,row2+1):   
            for j in range(col1,col2+1):
                self.rectangle[i][j] = newValue
        return self.rectangle 
    def getValue(self, row: int, col: int) -> int:
        value = self.rectangle[row][col]
        return value 
```
### 下一题解答中
