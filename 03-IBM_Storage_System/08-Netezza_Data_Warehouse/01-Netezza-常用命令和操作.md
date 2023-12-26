# Netezza-常用命令和操作
## 常用命令
命令`nzhw show -issues`查看硬件故障，示例：
```
[nz@nzhost ~]$ nzhw show -issues 
Description HW ID Location               Role     State       Security
----------- ----- ---------------------- -------- ----------- --------
Disk        1498  spa1.diskEncl11.disk21 Failed   Ok          Disabled
Disk        1526  spa1.diskEncl9.disk4   Failed   Ok          Disabled
```
官方参考链接：
- [Summary of command-line commands](https://www.ibm.com/docs/en/psfa/7.2.1?topic=cli-summary-command-line-commands)
- [The nzhw command](https://www.ibm.com/docs/en/SSULQD_7.2.1/com.ibm.nz.adm.doc/r_sysadm_nzhw_cmd.html)

## 常用操作
### 设备检查
### 手动failover硬盘
参考链接：[Manually failover a disk in Netezza](https://www.iexpertify.com/learn/manually-failover-a-disk-in-netezza/)
## 设备管理
### host管理
官方参考链接：[Netezza Manage hosts](https://www.ibm.com/docs/en/psfa/7.2.1?topic=tasks-manage-hosts)
### 硬盘管理
官方参考链接：[Netezza Manage disks](https://www.ibm.com/docs/en/psfa/7.2.1?topic=tasks-manage-disks)

## 常用维护
### 存储硬盘更换
Role为Failed存储硬盘更换步骤如下：
- 运行命令`nzhw -issues`查看故障硬盘ID，例如是1868
- 运行命令`nzhw -type disk |grep 1868`再次确认Role状态为`Failed`，状态为`Ok`
- 运行命令`nzhw locate -id 1868`点亮故障磁盘标识灯
- 物理拔掉故障硬盘，等几分钟
- 运行命令`nzhw -issues`查看Role状态为`Failed`，状态为`Missing`
- 运行命令`nzhw delete -id 1868`删掉硬盘
- 运行命令`nzhw -issues`查看故障磁盘不在了
- 安装新的磁盘
- 运行命令`nzhw -issues`查看新磁盘Role状态为`Inactive`，状态为`Ok`，记住新ID，例如是1777
- 运行命令`nzhw activate -id 1777`激活硬盘
- 运行命令`nzhw -type disk |grep 1777`查看Role状态为`Sparing`，状态为`Ok`
- 等一两分钟再次查看Role状态为`Spare`，状态为`Ok`
- 如果标识灯还在，运行命令`nzhw locate -id 1868 -off`关闭

### hostdisk维护
warning状态的hostdisk磁盘维护步骤如下：
- 运行命令`nzhw -issues`查看故障硬盘ID，例如是1669，以及热备盘ID，例如是1670
- 运行命令`nzhw -type hostdisk`查看热备使用情况，warning状态下磁盘热备不会顶上去使用，确认故障磁盘Role状态为`Active`，状态为`Ok`
- 运行命令`nzhw locate -id 1669`不会点亮故障磁盘标识灯，会说明位置，lower或upper的host的几号槽位：
    - 注意，槽位提示是1st，2nd，3rd，4th，5th这种，但是机器上槽位编号是0开始，0，1，2，3，4，5这种，要注意
- 找到位置拔掉故障硬盘，等一两分钟，故障硬盘槽位的橙色灯亮，热备磁盘1670的槽位上绿灯闪和橙色灯亮，同时有块数据盘绿色灯也亮了，raid在同步数据，服务器的告警灯也是亮的
- 此时使用命令`nzhw -issues`查看状态：
    - 热备盘1670，Role为`Assigning`，状态为`Ok`
    - 故障盘1669，Role为`Active`，状态为`Missing`
- 插入新的硬盘到原来1669磁盘的位置，插入后hostdisk ID不变
- 此时使用命令`nzhw -issues`查看状态：
    - 热备盘1670，Role为`Assigning`，状态为`Ok`
    - 新磁盘1669，Role为`Failed`，状态为`Warning`
- 此时1669磁盘未亮灯，热备磁盘1670的槽位上绿灯闪和橙色灯亮，同时有块数据盘绿色灯也亮了
- 等待热备磁盘1670磁盘数据同步完成，完成后：
    - 磁盘1670，Role为`Active`，状态为`Ok`
    - 磁盘1669，Role为`Assigning`，状态为`Ok`
    - 机器上1669磁盘绿灯闪和橙色灯亮，同时有块数据盘绿色灯也亮了
- 换上去的磁盘加入raid在进行数据同步了
- 等待完成，完成后：
    - 磁盘1670变为热备，Role为`Spare`，状态为`Ok`
    - 磁盘1669，Role为`Active`，状态为`Ok`
    - 机器上告警灯和硬盘上的橙色灯均熄灭
- 命令`nzhw -issues`查看硬件故障，已经没有hostdisk，更换完成

官方参考链接：
- [Host Disk Failure / SAS Controller Warning](https://www.ibm.com/support/pages/host-disk-failure-sas-controller-warning)
- [PureData System for Analytics: Host Disk 'Failed Down'](https://www.ibm.com/support/pages/puredata-system-analytics-host-disk-failed-down)
