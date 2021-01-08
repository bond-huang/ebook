# WAS-其它报错处理
遇到的一些报错的处理过程。
## WAS启动问题
### 启动WAS agent时候报错
报错代码：JVMJ9TI001E和JVMJ9VM015W，示例：
```
JVMJ9TI001E Agent library am_ibm_16 could not be opened (Could not load module .
System error: No such file or directory)
JVMJ9VM015W Initialization error for library j9jvmti29(-3): JVMJ9VM009E J9VMDllMain failed
```
原因：This is caused due to incorrect permissions or missing libraries in the toolkit dir.

IBM官方说明：[WAS agent fails with "JVMJ9TI001E Agent library am_ibm_16 could not be opened errors" when starting.](https://www.ibm.com/support/pages/node/6128037?mhsrc=ibmsearch_a&mhq=JVMJ9TI001E%20JVMJ9VM015W%20)

### WAS server启动时报错
SystemOut.log里面事件示例：
```
[21-1-7 19:45:10:458 CST] 0000006c wtp           E org.eclipse.jst.j2ee.commonarchivecore.internal.strategy.DirectoryLoadStrategyImpl addFileFromBinariesDirectory Failed to open archive [ WEB-INF/lib/poi-3.11.jar ]
[21-1-7 19:45:10:460 CST] 0000006c wtp           W org.eclipse.jst.j2ee.commonarchivecore.internal.impl.CommonarchiveFactoryImpl openNestedArchive An error occurred while opening a nested archive: error in opening zip file
[21-1-7 19:45:10:461 CST] 0000006c wtp           W org.eclipse.jst.j2ee.commonarchivecore.internal.impl.CommonarchiveFactoryImpl openNestedArchive Base exception
                                 java.util.zip.ZipException: error in opening zip file
	at java.util.zip.ZipFile.open(Native Method)
	at java.util.zip.ZipFile.<init>(ZipFile.java:150)
	at java.util.zip.ZipFile.<init>(ZipFile.java:166)
...
```
停止服务器并确认没有java进程后先清除一下缓存，删除下面示例目录中文件:
```
/home/wasusr/IAMServer01/temp/*
/home/wasusr/IAMServer01/wstemp/*
/home/wasusr/IAMServer01/configuration/*
/home/wasusr/IAMServer01/servers/BIMServer01/*
```
执行脚本：
```
/home/wasusr/IAMServer01/bin/osgiCfgInit.sh
/home/wasusr/IAMServer01/bin/clearClassCache.sh
```
然后再启动WAS。

SystemOut.log里如果还有如下报错：
```
Caused by: java.io.FileNotFoundException: /home/wasusr/IAMServer01/logs/ffdc/BIMServer01_exception.log (打开的文件过多)
```
代表当前OS的ulimit参数里nofiles值比较低，例如1024,建议增大至8096 (如果使用ulimit命令设置重新启动后会失效，查看修改/etc/limits文件)

如果启动server还有错，收集新的SystemErr.log和SystemOut.log，以及下面内容：
```
ulimit -a > ulimit.txt 
ls /home/wasusr/IBM/WebSphere/AppServer > ls.txt
```
## 待补充
