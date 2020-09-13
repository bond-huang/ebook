# Openpyxl-Excel基础操作
使用Openpyxl对Excel进行基础操作的学习笔记。
## 创建工作表
&#8195;&#8195;不需要在文件系统中创建文件就可以使用openpyxl，导入Workbook类并且实例化，然后通过`Workbook.active`来创建worksheet，示例如下：
```python
from openpyxl import Workbook
wb = Workbook()
ws =wb.active
```
可以使用`Workbook.create_sheet()`方法创建新的worksheet:
```python
# 插入到末尾（默认）
ws1 = wb.create_sheet("Mysheet")
# 在第一个位置插入
ws2 = wb.create_sheet("Mysheet",0)
# 在倒数第二个位置插入
ws3 = wb.create_sheet("Mysheet",-1)
```
默认情况下Sheets会自动命名，按顺序编号（Sheet，Sheet1，Sheet2等），可以随时通过`Worksheet.title`属性更改名称,以及：
```python
ws.title = "New Test"
```
默认情况下包含该标题的选项卡的背景颜色为白色。可以更改此属性RRGGBB，通过`Worksheet.sheet_properties.tabColor`来更改：
```python
ws.sheet_properties.tabColor = "1072BA"
```
给worksheet命名后，就可以将其作为workbook的key：
```python
w3 = wb["New test"]
```
使用`Workbook.sheetname`属性查看workbook中所有worksheet的名称,并且对worksheet进行遍历：
```python
print(wb.sheetnames)
for sheet in wb:
    print(sheet.title)
```
可以使用`Workbook.copy_worksheet()`方法在单个workbook中创建worksheet的副本：
```python
source = wb.active
target = wb.copy_worksheet(source)
print(target)
```
以上所有代码整合到一起运行后结果示例如下：
```
PS C:\Users\Desktop\big1000\vscode\codefile\openpyxl> python 1.py
['Mysheet1', 'New Test', 'Mysheet2', 'Mysheet']
Mysheet1
New Test
Mysheet2
Mysheet
<Worksheet "Mysheet1 Copy">
```
说明：
- 创建副本仅复制单元格（包括值，样式，超链接和注释）和某些工作表属性（包括尺寸，格式和属性），不复制所有其他工作簿/工作表属性，例如图像，图表等
- 也不能在工作簿之间复制工作表；如果工作簿是只读的或以只读模式打开的，则不能复制工作表

## 数据操作
### 访问一个cell
访问一个cell基本操作：
```python
# 单元格可以直接作为worksheet的key进行访问
a = ws['A4']
# 如果单元格不存在则创建一个单元格，可以直接分配值
ws['A4'] = 10
# 使用Worksheet.cell()方法指定行和列来访问单元格
b = ws.cell(row=4,clounm=2,value=100)
```
&#8195;&#8195;注意：在内存中创建工作表时，它不包含任何单元格，它们只是在首次访问时创建的。即使不为它们分配值，在内存中全部创建它们时候可以滚动浏览而不是直接访问它们，例如用一个for循环在内存中创建100x100的空单元格：
```python
for i in range(1,101):
    for j in range(1,101):
        ws.cell(row=i,column=j)
```
### 访问多个cell
使用切片及使用行和列范围访问单元格范围：
```python
cell_range = ws['B1':'D5']
colE = ws['E']
col_range = ws['A:E']
row9 = ws[9]
row_range = ws[3:8]
```
也可以使用`Worksheet.iter_rows()`方法，示例：
```python
for row in ws.iter_rows(min_row=1,max_col=2,max_row=2):
    for cell in row:
        print(cell)
```
运行示例如下：
```
PS C:\Users\Desktop\big1000\vscode\codefile\openpyxl> python 1.py
<Cell 'New Test'.A1>
<Cell 'New Test'.B1>
<Cell 'New Test'.C1>
<Cell 'New Test'.A2>
<Cell 'New Test'.B2>
<Cell 'New Test'.C2>
```
同样`Worksheet.iter_cols()`方法将返回列：
```python
for col in ws.iter_cols(min_row=1,max_col=3,max_row=2):
    for cell in col:
        print(cell)
```
运行示例如下：
```
PS C:\Users\Desktop\big1000\vscode\codefile\openpyxl> python 1.py
<Cell 'New Test'.A1>
<Cell 'New Test'.A2>
<Cell 'New Test'.B1>
<Cell 'New Test'.B2>
<Cell 'New Test'.C1>
<Cell 'New Test'.C2>
```
注意：由于性能原因，方法`Worksheet.iter_cols()`在只读模式下不可用。

`Worksheet.rows`可以遍历文件的所有行或列：
```python
ws = wb.active
ws['C7'] = 'Thanos'
print(tuple(ws.rows))
```
运行示例如下：
```
((<Cell 'Sheet'.A1>, <Cell 'Sheet'.B1>, <Cell 'Sheet'.C1>),
 (<Cell 'Sheet'.A2>, <Cell 'Sheet'.B2>, <Cell 'Sheet'.C2>), 
 (<Cell 'Sheet'.A3>, <Cell 'Sheet'.B3>, <Cell 'Sheet'.C3>), 
 (<Cell 'Sheet'.A4>, <Cell 'Sheet'.B4>, <Cell 'Sheet'.C4>), 
 (<Cell 'Sheet'.A5>, <Cell 'Sheet'.B5>, <Cell 'Sheet'.C5>), 
 (<Cell 'Sheet'.A6>, <Cell 'Sheet'.B6>, <Cell 'Sheet'.C6>), 
 (<Cell 'Sheet'.A7>, <Cell 'Sheet'.B7>, <Cell 'Sheet'.C7>))
```
或Worksheet.columns属性：
```python
ws = wb.active
ws['C4'] = 'Thanos'
print(tuple(ws.columns))
```
运行示例如下：
```
((<Cell 'Sheet'.A1>, <Cell 'Sheet'.A2>, <Cell 'Sheet'.A3>, <Cell 'Sheet'.A4>), 
(<Cell 'Sheet'.B1>, <Cell 'Sheet'.B2>, <Cell 'Sheet'.B3>, <Cell 'Sheet'.B4>), 
(<Cell 'Sheet'.C1>, <Cell 'Sheet'.C2>, <Cell 'Sheet'.C3>, <Cell 'Sheet'.C4>))
```
注意：由于性能原因，方法`Worksheet.columns`在只读模式下不可用。

### 读取值
&#8195;&#8195;如果需要读取worksheet中的值，则可以使用`Worksheet.values`,会遍历worksheet中的所有行，但仅返回单元格值,示例如下：
```python
for row in ws.values:
    for values in row:
        print(value)
```
运行示例如下：
```
None
None
None
None
None
None
None
None
None
None
None
Thanos
```
可以在`Worksheet.iter_rows()`和`Worksheet.iter_cols()`中采用`values_only`参数，只返回单元格的值：
```python
for row in ws.iter_rows(min_row=1,max_col=3,max_row=4,values_only=True):
    print(row)
```
运行示例如下：
```
(None, None, None)
(None, None, None)
(None, None, None)
(None, None, 'Thanos')
```
## 存储数据
worksheet中有了cell，就可以给其分配值：
```python
a = ws['A4']
a.value = 'Miracles happen every day !'
print(a.value)
b = ws.cell(row=4,column=2)
b.value = '3.1415926'
print(b.value)
```
运行示例如下：
```
PS C:\Users\Desktop\big1000\vscode\codefile\openpyxl> python 1.py
Miracles happen every day !
3.1415926
```
### 保存到文件
保存workbook的最简单，最安全的方法是使用对象`Workbook`中的 `Workbook.save()`方法：
```python
wb = Workbook()
wb.save('test.xlsx')
```
说明：
- 此操作将覆盖现有文件，并且不会有提示
- 文件扩展名不是强制为xlsx或xlsm，不写也可以，但是使用过程中可能会比较麻烦
- 由于OOXML文件基本上是ZIP文件，因此可以使用相关ZIP玩家管理器将其打开使用

### 另存为流
如果要将文件保存到流中，例如在使用Web应用程序时，可以使用`NamedTemporaryFile()`,示例如下：
```python
from tempfile import NamedTemporaryFile
from openpyxl import Workbook
wb = Workbook()
with NamedTemporaryFile() as tmp:
    wb.save(tmp.name)
    tmp.seek(0)
    stream = tmp.read()
```
可以指定属性`template = True`将workbook另存为模板：
```python
wb = load_workbook('document.xlsx')
wb.template = True
wb.save('document_template.xltx')
```
或将此属性设置为False（默认），以另存为文档：
```python
wb = load_workbook('document_template.xltx')
wb.template = False
wb.save('document.xlsx', as_template=False)
```
注意：注意数据属性和文档扩展名以将文档保存在文档模板中，否则表引擎可能无法打开文档。

## 从文件加载
可以使用`openpyxl.load_workbook()`打开现有的workbook：
```python
from openpyxl import load_workbook
wb2 = load_workbook('test.xlsx')
print wb2.sheetnames
```
运行示例如下：
```
PS C:\Users\Desktop\big1000\vscode\codefile\openpyxl> python 2.py
['Sheet', 'test1', 'Newsheet']
```
