# Element_Plus-学习笔记.
记录学习和使用过程中知识点，避免忘记。
## Table表格组件
官方参考文档：[https://element-plus.gitee.io/#/zh-CN/component/table](https://element-plus.gitee.io/#/zh-CN/component/table)

### 合并行或列
#### 官方示例
官方文档中，展示了两个示例，分别是列合并和行合并，官方示例中HTML代码：
```v
<el-table
  :data="tableData"
  :span-method="arraySpanMethod"
  border
  style="width: 100%">

<el-table
  :data="tableData"
  :span-method="objectSpanMethod"
  border
  style="width: 100%; margin-top: 20px">
```
JavaScript代码：
```js
methods: {
    arraySpanMethod({ row, column, rowIndex, columnIndex }) {
        if (rowIndex % 2 === 0) {
          if (columnIndex === 0) {
            return [1, 2];
          } else if (columnIndex === 1) {
            return [0, 0];
          }
        }
      },
    objectSpanMethod({ row, column, rowIndex, columnIndex }) {
        if (columnIndex === 0) {
          if (rowIndex % 2 === 0) {
            return {
              rowspan: 2,
              colspan: 1
            };
          } else {
            return {
              rowspan: 0,
              colspan: 0
            };
          }
        }
      }
   }
```
说明：
- 官方示例中使用了`arraySpanMethod`和`objectSpanMethod`分别进行和列合并，合并的条件进行了多次判断，具体效果参考官方文档
- 通过给`table`传入`span-method`方法可以实现合并行或列，方法的参数是一个对象，里面包含当前行`row`、当前列`column`、当前行号`rowIndex`、当前列号`columnIndex`四个属性
- 该函数可以返回一个包含两个元素的数组，第一个元素代表`rowspan`，第二个元素代表`colspan`,例如`arraySpanMethod`中返回的数组；也可以返回一个键名为`rowspan`和`colspan`的对象，例如`objectSpanMethod`中返回的对象

#### 使用示例
&#8195;&#8195;官方文档只是演示功能，实际使用需求肯定不一样，编写不同的JavaSript实现不同功能。我的需求就只是想把每一行的第五列和第六列合并，表头还是分开的，实现很简单，但个人对JavaSript不熟，所以看懂官方说明后，记录下来避免忘记。

在HTML代码中：
```
<el-table
  :data="tableData"
  :span-method="arraySpanMethod"
  border>
```
JavaScript代码：
```js
methods: {
  arraySpanMethod({ columnIndex }) {
    if (columnIndex === 4) {
      return [1, 2];
    }
  },
}
```
说明：
- 使用`border`后有表格边界，方便查看是否合并了，不需要可以删除
- 示例中只使用到了`columnIndex`，其它`row, column, rowIndex`等没使用就不用写，写了还会报错：`'row' is declared but its value is never read.Vetur(6133)`
- 返回`[1, 2]`代表`rowspan`为1，`colspan`为2，代表意思和HTML5中`td`标签一样，用法也一样，此值在HTML基础学习中有相应记录：[HTML-基础学习笔记](https://ebook.big1000.com/08-Python/07-Python_Flask/11-HTML-%E5%9F%BA%E7%A1%80%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0.html?h=rowspan)

## 待补充
