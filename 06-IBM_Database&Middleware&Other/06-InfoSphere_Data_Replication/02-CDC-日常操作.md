# CDC-日常操作
## IBM i系统日常操作
### 设置断点
设置断点有两种方式，时间为例：
```
SETJRNPOS JOURNAL(TMPJRNLIB/TMPJRN) TARGET(TMP1) JRNRCVNME(TMPJRNLIB1/TMPJRN4159) STRDTE(03232024) STRTIM(233000) OPEN(*NO)
```
具体日志点示例：
```
SETJRNPOS JOURNAL(TMPJRNLIB/TMPJRN) TARGET(TMP1) JRNRCVNME(TMPJRNLIB1/TMPJRN4159) STRSEQNBR(00000001397738453635)
```
注意：TARGET是订阅名称，具体订阅名称可以在CDC软件管理APP上看到，或者AS400里面CDC软件库里面也有。

参考链接：
- [SETJRNPOS - Set journal position](https://www.ibm.com/docs/en/idr/11.4.0?topic=commands-setjrnpos-set-journal-position)
- [When Mark capture point and SETJRNPOS command are combined to use](https://www.ibm.com/support/pages/when-mark-capture-point-and-setjrnpos-command-are-combined-use)

## 待补充
