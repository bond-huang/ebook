# SVC-固件升级
即SVC的微码版本升级，官方参考文档：
- [Software Upgrade Test Utility](https://www.ibm.com/support/pages/node/651127)
## 升级前准备
准备工作主要是健康健康、配置备份即升级测试。
### 健康检查
检查内容：
- 检查是否有需要维护的事件
- 检查外部存储器、mdisk和vdisk是否都正常
- 检查主机访问都是否正常，远程复制等是否正常
- 检查管理IP及各节点服务IP是否正常访问
- 检查光纤端口是否都正常
- 检查电池是否online或charged
- 检查SVC性能是否正常

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
GUI升级很简单，不作过多记录，使用CLI方式简单记录下：
