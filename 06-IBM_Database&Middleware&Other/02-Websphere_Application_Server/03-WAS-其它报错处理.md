# WAS-其它报错处理
遇到的一些报错的处理过程。

### 启动WAS agent时候报错
报错代码：JVMJ9TI001E和JVMJ9VM015W，示例：
```
JVMJ9TI001E Agent library am_ibm_16 could not be opened (Could not load module .
System error: No such file or directory)
JVMJ9VM015W Initialization error for library j9jvmti29(-3): JVMJ9VM009E J9VMDllMain failed
```
原因：This is caused due to incorrect permissions or missing libraries in the toolkit dir.

IBM官方说明：[WAS agent fails with "JVMJ9TI001E Agent library am_ibm_16 could not be opened errors" when starting.](https://www.ibm.com/support/pages/node/6128037?mhsrc=ibmsearch_a&mhq=JVMJ9TI001E%20JVMJ9VM015W%20)

### 待补充
