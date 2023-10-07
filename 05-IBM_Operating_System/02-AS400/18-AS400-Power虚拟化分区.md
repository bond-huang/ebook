# AS400-Power虚拟化分区
IBM i虚拟化主要有iHost和PowerVM。
## PowerVM
主机相关链接：
- [IBM Power9 Virtual I/O Server](https://www.ibm.com/docs/en/power9/9080-M9S?topic=environment-virtual-io-server)
- [IBM Power Virtualization Center (PowerVC) for IBM i](https://www.ibm.com/support/pages/ibm-power-virtualization-center-powervc-ibm-i)
- [Partitioning with a IBM i](https://www.ibm.com/docs/zh/i/7.3?topic=partitions-partitioning-i)
- [IBM i 7.3 Virtual Partition Manager](https://www.ibm.com/docs/zh/i/7.3?topic=i-virtual-partition-manager)
- [IBM i 7.3 Miscellaneous limits](https://www.ibm.com/docs/zh/i/7.3?topic=capacities-miscellaneous-limits)
- [IBM i 客户机分区注意事项](https://www.ibm.com/docs/zh/i/7.3?topic=software-i-client-partition-considerations)
- [在集成虚拟化管理器(IVM)受管服务器上的Virtual I/O Server(VIOS)中配置高级节点故障检测](https://www.ibm.com/docs/zh/i/7.3?topic=canfd-configuring-advanced-node-failure-detection-in-virtual-io-server-vios-integrated-virtualization-manager-ivm-managed-server)

存储相关链接：
- [IBM FlashSystem 9x00 Environments for IBM i hosts](https://www.ibm.com/docs/en/flashsystem-9x00/8.3.x?topic=arih-environments-i-hosts)
- [IBM FlashSystem 9x00 Configuring the IBM i operating system](https://www.ibm.com/docs/en/flashsystem-9x00/8.3.x?topic=server-configuring-i-operating-system)
- [FlashSystem 9x00 IBM Power Systems with Virtual I/O Server](https://www.ibm.com/docs/en/flashsystem-9x00/8.3.x?topic=attachments-power-systems-virtual-io-server)
- [FlashSystem 9x00 Known IBM i issues and limitations](https://www.ibm.com/docs/en/flashsystem-9x00/8.3.x?topic=psvis-known-i-issues-limitations)
- [FlashSystem 9x00 Attachment requirements for IBM i hosts](https://www.ibm.com/docs/en/flashsystem-9x00/8.3.x?topic=psvis-attachment-requirements-i-hosts)
- [DS8900 Configurations for IBM Power Systems hosts running IBM i](https://www.ibm.com/docs/zh/ds8900/9.0.2?topic=i-configurations-power-systems-hosts-running)

### 配置限制条件
#### 主机上虚拟化相关限制条件
如下表(IBM i7.3)：     
限制条件|说明
:---|:---
VSCSI光驱的最大数量|16
VSCSI磁带设备的最大数量|4
VSCSI磁盘设备的最大数量|32
每个NPIV的最大活动存储卷路径数|7.3TR7或更高版本为127，其它64
虚拟媒体设备的最大大小|1000000MB
虚拟磁带资源的最大数量|35
最大虚拟光资源|35

#### 存储上相关限制  
最小磁盘资源限制：
- DS8000或使用VIOS作为服务器的虚拟磁盘：520字节扇区需要35GB
- SVC、Storwize、使用VIOS VSCSI或IBM i作为服务器的虚拟磁盘：40 GB（35GB可用空间）
- 本机连接的SAS/SCSI（520字节或4160字节扇区）：70 GB

最大磁盘资源限制：
- 512/520 block disk: 2TB减去1 block
- 4096 block disk: 2TB减去1 block
- 4196 block disk: 4TB减去1 block

### 待补充

