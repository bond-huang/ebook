# SVC-故障排查及处理

## LED灯状态
### SVC前部LED灯
各种型号LED灯官方说明：
- [SAN Volume Controller 2145-DH8 front panel controls and indicators](https://www.ibm.com/docs/en/sanvolumecontroller/8.3.x?topic=nci-san-volume-controller-2145-dh8-front-panel-controls-indicators)
- [SAN Volume Controller 2145-SV1 front panel controls and indicators](https://www.ibm.com/docs/en/sanvolumecontroller/8.3.x?topic=nci-san-volume-controller-2145-sv1-front-panel-controls-indicators)

## 配置问题
### 配置备份与恢复
#### 配置备份
可以通过以下命令进行备份当前配置：
```
IBM_2145:SVC_Cluster:superuser>svcconfig backup
```
将备份文件到自己电脑：
```sh
### (Using Linux) ###
scp superuser@cluster_ip:/dumps/svc.config.backup.* .
### (Using Windows) ###
pscp -unsafe superuser@cluster_ip:/dumps/svc.config.backup.* .
```
#### 配置恢复
官方参考链接：[Restoring the system configuration](https://www.ibm.com/docs/en/sanvolumecontroller/8.6.x?topic=configuration-restoring-system)

## 硬件维护
### 主板更换
各种型号主板更换官方说明：
- [Replacing the system board: 2145-DH8](https://www.ibm.com/docs/en/sanvolumecontroller/8.2.x?topic=board-replacing-system-2145-dh8)
- [Replacing the main board: 2145-SV1](https://www.ibm.com/docs/en/sanvolumecontroller/8.2.x?topic=board-replacing-main-2145-sv1)

## 引导盘问题
### 引导盘
官方参考链接：[Resolving a problem with the SAN Volume Controller boot drives](https://www.ibm.com/docs/en/sanvolumecontroller/8.2.x?topic=problem-resolving-san-volume-controller-boot-drives)
## SAN网络问题
### NPIV问题
官方参考链接：[Troubleshooting SVC/Storwize NPIV Connectivity](https://www.ibm.com/support/pages/troubleshooting-svcstorwize-npiv-connectivity#:~:text=One%20of%20the%20most%20common%20problems%20that%20I,of%20these%20vWWPNs.%20lsportfc%20will%20list%20the%20pWWPNs.)

## 数据拷贝问题
### 引导盘拷贝问题
&#8195;&#8195;SVC上的系统引导盘，右键直接复制，然后映射给AIX分区进行引导，但是怎么也扫描不到磁盘。后来发现不能直接右键复制再映射给其它分区，需要在FlashCopy选项里面点击创建克隆选项，命令完成后不用等后台完成即可映射给新的分区使用。
## 待补充