# RPG-基础知识
## RPG基础
[IBM i 7.1 Program Status Codes](https://www.ibm.com/docs/en/i/7.1?topic=structure-program-status-codes)

## AS400程序中时间
参考链接：
- [IBM i 7.2 Date Data Type](https://www.ibm.com/docs/en/i/7.2?topic=formats-date-data-type)
- [IBM i 7.2 TEST (Test Date/Time/Timestamp)](https://www.ibm.com/docs/en/i/7.2?topic=codes-test-test-datetimetimestamp#zztest)
- [IBM i 7.2 Moving Date-Time Data](https://www.ibm.com/docs/en/i/7.2?topic=operations-moving-date-time-data)
- [IBM i 7.3 Prevention of date, time, and timestamp errors when copying files](https://www.ibm.com/docs/en/i/7.3?topic=pewcf-prevention-date-time-timestamp-errors-when-copying-files&mhsrc=ibmsearch_a&mhq=IBM%20I%202039)
- [IBM i 7.2 Examples of Converting a Character Field to a Date Field](https://www.ibm.com/docs/en/i/7.2?topic=data-examples-converting-character-field-date-field)

&#8195;&#8195;RPG程序中如果不特别指定，默认情况下年份在40到99之间，则假设2位年份日期字段或值的世纪为19；如果年份在00到39之间，则假定世纪为20。    
&#8195;&#8195;日期变量的默认内部格式为`*ISO`。此默认内部格式可以由控制规范关键字DATFMT全局覆盖，由`/SET`和`/RESTORE`指令临时更改，并由定义规范关键字`DATE`或`DATFMT`单独设置。

确定日期字段的内部日期格式和分隔符时使用的层次结构为
- 从定义规范中指定的`DATE`或`DATFMT`关键字
- 来自最新的`/SET`指令，该指令包含尚未由`/RESTORE`指令还原的`DATFMT`关键字
- 从控制规范中指定的`DATFMT`关键字
- `*ISO`

&#8195;&#8195;根据可以表示的年份范围，有三种日期数据格式。当操作的结果是目标字段的有效范围之外的日期时，可能会出现日期溢出或下溢情况。格式和范围如下：

一年中的位数|年份范围
:---|:---
2 (*YMD, *DMY, *MDY, *JUL)|1940到2039
3 (*CYMD, *CDMY, *CMDY)|1900到2899
4 (*ISO, *USA, *EUR, *JIS, *LONGJUL)|0001到9999

## 待补充