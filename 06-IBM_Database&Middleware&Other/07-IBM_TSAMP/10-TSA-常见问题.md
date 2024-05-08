# TSA-常见问题
## 启停问题
### 停止一直Pending
停止TSA资源组时候，某个应用一直在Pending状态：
- 应用停止脚本运行异常，没能停掉相关应用，手动停止即可
- 有其他进程占用了TSA的相关资源，导致无法停止，查找相关进程

运行命令查看应用相关启停脚本：
```sh
lsrsrc -l IBM.Application
```
## 节点状态问题
### 节点状态Failed Offline
&#8195;&#8195;lssam命令输出看到某节点资源状态是`Failed Offline`，通常是由于节点异常宕机导致，并且此时如果进行切换，会切换失败。重启节点可以解决，或者运行命令：
```shell
resetrsrc –s ‘Name=”<resource_name>” && NodeNameList={“node_name”}’ IBM.Application
```
官方参考链接：[A resource has an OpState of Failed Offline](https://www.ibm.com/docs/en/tsafm/4.1.0?topic=analysis-resource-has-opstate-failed-offline)
## 其它
### lssam命令2612-023报错
lssam命令输出提示：
```
(lsrsrc-api) /usr/sbin/rsct/bin/lsrsrc-api：2612-023 找不到资源。 
lssam：出现意外的 RMC 错误。RMC 返回码为 1。
```
语言环境的原因，官方参考链接：[IV74701: LSSAM COMMAND OUTPUT ERROR 2612-023](https://www.ibm.com/support/pages/apar/IV74701)

## 待补充