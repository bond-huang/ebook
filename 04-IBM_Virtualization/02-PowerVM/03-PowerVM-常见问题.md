# PowerVM-常见问题
VIOS系统中常见问题。
## HMC连接问题
### HMC查看虚拟网络报错
#### 报错示例一
报错示例：
```
"Error occurred while querying for SharedEthernetAdapter from VIOS <VIOS_PARTITION_NAME> with ID <VIOS_PARTITION_ID) in System <MANAGED_SYSTEM_NAME>- Unable to connect to Database."
```
官方描述：      
[HMC Enhanced GUI-"Error occurred while querying for SharedEthernetAdapter... Unable to connect to Database"](https://www.ibm.com/support/pages/node/961332?mhsrc=ibmsearch_a&mhq=error%20occurred%20while%20querying%20for%20sharedethernetadapter%20from%20vios)

#### 报错示例二
报错示例：
```
Error occurred while querying for SharedEthernetAdapter from VIOS <PARTITION_NAME> with ID <PARTITION ID> in System <MACHINE_TYPE-MODEL*SERIAL_NO> - The system is currently too busy to complete the specified request. Please retry the operation at a later time. If the operation continues to fail, check the error log to see if the filesystem is full.
```
官方说明：    
[HMC Enhanced GUI - "Error occurred while querying for SharedEthernetAdapter...The system is currently too busy to complete the specified request."](https://www.ibm.com/support/pages/node/1073894?mhsrc=ibmsearch_a&mhq=error%20occurred%20while%20querying%20for%20sharedethernetadapter%20from%20vios)

### 查看存储池报错
报错示例：
```
"Cannot connect to one or more Virtual I/O Servers.
Virtual I/O Server Error Details
Error occurred while querying for VirtualMediaRepository from VIOS <VIOS_PARTITION_NAME> with ID <VIOS_PARTITION_ID) in System <MANAGED_SYSTEM_NAME>- Unable to connect to Database."
```
官方描述：   
[HMC Enhanced GUI - "Error occurred while querying for VirtualMediaRepository from VIOS... Unable to connect to Database."](https://www.ibm.com/support/pages/node/1077855?mhsrc=ibmsearch_a&mhq=error%20occurred%20while%20querying%20for%20sharedethernetadapter%20from%20vios)

### 解决方法
官方针对以上问题的解决方案：        
[When Using HMC GUI you see message "Unable to connect to the Database Error occurred "](https://www.ibm.com/support/pages/when-using-hmc-gui-you-see-message-unable-connect-database-error-occurred)
