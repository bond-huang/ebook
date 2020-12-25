# Python运维学习笔记-文档与报告
学习教材：《Python Linux系统管理与自动化运维》（赖明星著）
## 使用Python处理Excel文档
教程中使用的是openpyxl，之前有专门学习过，回头继续学习。
## 使用Python操作PDF文档
之前打算学习reportlab，但是发现AIX中安装不了，学习下课本中的PyPDF2。    
官方文档：[https://pythonhosted.org/PyPDF2/index.html](https://pythonhosted.org/PyPDF2/index.html)
### PyPDF2安装与介绍
PyPDF2是一个开源的库，需要先安装：
```
[root@redhat8 pdf]# pip install PyPDF2
```
### 使用PdfFileReader读取PDF文件
随便找个PDF文件：IBM i Basic printing.pdf，示例如下：
```
>>> import PyPDF2
f.py:1736]>>> reader = PyPDF2.PdfFileReader(open('IBM i Basic printing.pdf','rb'))
>>> reader.getNumPages()
424
>>> reader.getIsEncrypted()
False
>>> page = reader.getPage(10)
>>> page.extractText()
'2.If print spooling is selected, the output data is placed in a spooled file and the spoo
led file is placed\nin an output queue. If dir\nect printing is selected, the output data is sent dir\nectly to the printer\n.
...
>>> reader.getDocumentInfo()
{'/Creator': 'XPP', '/Keywords': '', '/Subject': '', '/Author': 'IBM', '/Title': 'IBM i: B
asic printing', '/CreationDate': "D:20180804081225-05'00'", '/ModDate': "D:20180804081225-05'00'", '/Producer': 'PDFlib+PDI 9.0.5 (AIX)'}
```
示例说明：
- 示例中首先使用二进制读模式打开PDF文件，并传递给PdfFileReader类的初始化函数
- PdfFileReader类的初始化函数会返回一个PdfFileReader类的对象，可以通过此对象来获取PDF文件内容
- 使用getNumPages函数获取了文件页数，使用getIsEncrypted查看是否加密
- 通过传递下标给getPage的方式获取PDF的页面，在PyPDF2中下标从0开始，所以getPage(10)是第十一页内容
- PdfFileReader类的getPage函数会返一个PageObject对象，对象中有旋转页面的rotateClockwise方法，合并页面的mergePage方法，示例中使用extractText方法提取页面中的文本
- 最后通过PdfFileReader的getDocumentInfo方法获取PDF文件的元信息

### 
