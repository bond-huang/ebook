# SVC-固件升级
SVC的微码版本升级，官方参考文档：
- [Software Upgrade Test Utility](https://www.ibm.com/support/pages/node/651127)
- [Updating the software automatically using the CLI](https://www.ibm.com/docs/en/sanvolumecontroller/7.8.1?topic=system-updating-software-automatically-using-cli)
- [Concurrent Compatibility and Code Cross Reference for Spectrum Virtualize](https://www.ibm.com/support/pages/node/5692850)

## 升级前准备
准备工作主要是健康健康、配置备份即升级测试。
### 健康检查
检查内容：
- 检查是否有需要维护的事件
- 检查外部存储器、mdisk和vdisk是否都正常，是否有脱机或降级的卷
- 如果有脱机的卷，`fast_write_state`如果为空，则卷可以处于脱机状态，在更新期间不会导致错误
- 检查主机访问都是否正常，远程复制等是否正常
- 检查管理IP及各节点服务IP是否正常访问
- 检查光纤端口是否都正常
- 检查电池是否online或charged
- 检查SVC性能是否正常
- 检查每个节点是否有依赖的卷

### 配置备份
首先收集一份系统日志备用，使用WEB界面收集即可。

可以通过以下命令进行备份当前配置：
```
IBM_2145:SVC_Cluster:superuser>svcconfig backup
```
通过SCP将备份文件到自己电脑：
```sh
### (Using Linux)###
scp superuser@cluster_ip:/dumps/svc.config.backup.* .
### (Using Windows)###
pscp -unsafe superuser@cluster_ip:/dumps/svc.config.backup.* .
```
### 升级测试
&#8195;&#8195;通过GUI测试很简单，不作过多记录，各版本界面操作方式有所不同，具体参考[Software Upgrade Test Utility](https://www.ibm.com/support/pages/node/651127)。使用CLI方式简单记录下。

先使用scp将文件拷贝到SVC上，示例如下：
```
# scp IBM2145_INSTALL_upgradetest_31.23 superuser@192.168.1.100:/upgrade/
superuser@10.8.254.60's password:
IBM2145_INSTALL_upgradetest_31.23                                                             100%  332KB  28.1MB/s   00:00
```
然后安装测试包：
```
IBM_2145:SVC_Cluster:superuser>svctask applysoftware -file IBM2145_INSTALL_upgradetest_31.23
CMMVC6227I The package installed successfully.
```
运行测试示例（后面版本是升级测试目标版本）：
```
IBM_2145:SVC_Cluster1:superuser>svcupgradetest -v 7.4.0.11
svcupgradetest version 31.23
Please wait, the test may take several minutes to complete.
******************* Warning found *******************
This upgrade introduces a new cache architecture.
For the duration of the upgrade the cache will be disabled
for the entire system, which may affect performance.
Once this upgrade has completed the cache will be automatically
re-enabled.

This system-wide cache disable is only required when installing
the new cache architecture for the first time.
Subsequent upgrades will not disable the cache on the entire system.
...
******************* Warning found *******************
This system is exposed to APAR HU02213, which can cause warmstarts in a remote system during upgrade.
Please see the following web page for details before continuing:
https://www.ibm.com/support/pages/node/6356439

Results of running svcupgradetest:
==================================
The tool has found 0 errors and 4 warnings.
None of these issues will prevent the software upgrade from being started.
Please review any warnings and errors to make sure it is safe to upgrade.
```
示例说明：
- 此次示例中发现0 errors，如果没有errors表示升级可以进行
- 此次示例中发现4 warnings，需要注意查看warnings说明，例如第一个就说会有新的缓存体系，升级会影响性能
- SVC一般没有内置硬盘，如果有或者是V7000等设备有内置硬盘，硬盘微码版本过低或者有bug也会在此提示，先升级硬盘微码方可继续

## 升级操作
### 使用WEB GUI升级
&#8195;&#8195;GUI升级很简单，不作过多记录，各版本界面操作方式有所不同，具体参考[Updating the software automatically using the CLI](https://www.ibm.com/docs/en/sanvolumecontroller/7.8.1?topic=system-updating-software-automatically-using-cli)。使用CLI方式简单记录下。

### 使用自动更新
首先通过scp或pscp微码文件拷贝到系统中：
```
# scp IBM2145_INSTALL_7.4.0.11 superuser@192.168.1.100:/upgrade/
```
在之前的测试步骤通过后，运行命令执行升级：
```
IBM_2145:SVC_Cluster:superuser>applysoftware -file IBM2145_INSTALL_7.4.0.11
```
如果是7.4.0之前的版本更新，使用如下CLI命令检查更新过程的状态：
```
svcinfo lssoftwareupgradestatus
```
说明：
- 该命令显示`inactive`，表示更新完成
- 如果状态为`stalled_non_redundant`，继续进行其余的节点更新集可能会导致脱机卷

如果是7.4.0或更高版本进行更新，使用如下CLI命令以检查更新过程的状态：
```
lsupdate
```
说明：
- 该命令显示`success`表示更新完成
- 如果状态为`stalled_non_redundant`，继续进行其余的节点更新集可能会导致脱机卷

&#8195;&#8195;如果从7.4.0之前的版本进行更新，则会收到状态消息`system_completion_required`。要完成更新过程，执行命令`applysoftware -complete`。运行该命令后，可以运行`lsupdate`来查看更新完成的进度。

### 验证是否成功
&#8195;&#8195;要验证更新是否成功完成，可以在GUI中查看，或者在系统中的每个节点执行`lsnodevpd`CLI命令。代码版本字段显示新的代码级别表示更新成功。

### 手动更新系统
个版本有所不一样，7.8版本官方参考文档：[Updating the system manually](https://www.ibm.com/docs/en/sanvolumecontroller/7.8.1?topic=system-updating-manually)

## 注意事项
注意事项说明如下：
- 注意根据官方Code Cross Reference来制定升级方案，版本跨度大可能需要中间版本作为跳板
- 升级前检查中如果有脱机的卷，并且`fast_write_state`为空，则卷可以处于脱机状态，在更新期间不会导致错误
- 在更新过程中任何节点遇到内存DIMM故障，立即停止并按照更新系统中的说明进行操作
- 部分版本升级会引入了新的缓存体系结构，集群缓存功能会被关闭，升级过程中可能会影响性能
- 节点更新之间有30分钟的延迟，延迟使主机多路径软件有时间重新发现到更新节点的路径
- 每个节点可能需要30多分钟进行系统自动更新
