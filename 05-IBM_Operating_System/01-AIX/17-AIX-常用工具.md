# AIX-常用工具
## AIX系统层常用工具
### AIX支持工具
AIX Support工具官方说明：[AIX Support Center Tools](https://www.ibm.com/support/pages/aix-support-center-tools)
#### summ诊断工具
summ诊断工具官方参考链接：[IBM AIX Diagnostic Tool "summ": A Summarized System Error log and Report Generator for I/O devices.](https://www.ibm.com/support/pages/ibm-aix-diagnostic-tool-summ-summarized-system-error-log-and-report-generator-io-devices)

下载后安装示例（或者/usr/local/bin没有就/usr/bin）：
```
# cd /usr/local/bin

# tar -xvf /tmp/summ_version_n.tar 

# chmod 755 summ
```
使用示例：
```
# errpt -a > errpt.out
# summ errpt.out
```
或者：
```
# errpt -a | summ
```
&#8195;&#8195;本机不方便，可以把snap日志里面的`errpt.out`可以拷贝到其他系统，或者直接`errpt -a > errpt.out`输出`errpt.out`,使用示例：
```sh
# summ errpt.out
Aug 16 23:29:48 hdisk1     T SC_DISK_ERR4        path  5 WRITE(10)        (00000002,0001) command timeout
Aug 16 23:29:47 hdisk0     T SC_DISK_ERR4        path  4 READ(10)         (369D2200,0100) command timeout
Aug 16 23:29:46 hdisk0     I SC_DISK_PCM_ERR9    path  4 path recovered
Aug 16 23:29:46 fcs6       N NONE
Aug 16 23:29:25 fscsi6     T FCP_ERR4            Cancel completed but task mgmt cancel timed out for port 0xC40101; re-issuing cancel
Aug 16 23:29:24 fscsi6     T FCP_ERR4            Cancel completed but task mgmt cancel timed out for port 0xC40001; re-issuing cancel
Aug 16 23:29:24 fscsi6     T FCP_ERR4            Cancel completed but task mgmt cancel timed out for port 0xC40201; re-issuing cancel
Aug 16 23:28:27 hdisk0     P SC_DISK_ERR7        path  4 path failure; READ(10)         (369D2200,0100) command timeout
```
## 其他平台使用工具