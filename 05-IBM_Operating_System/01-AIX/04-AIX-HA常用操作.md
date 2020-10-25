# AIX-HA常用操作
PowerHA是AIX系统中常用的高可用软件。
## 日志查看和收集
### 日志查看
大多数HA日志存放在/var/hacmp目录下，常用的有：
- 命令`errpt -e`输出：系统日志，会有HA集群事件
- /var/hacmp/adm/cluster.log：包含由PowerHA SystemMirror脚本和守护程序生成的带时间戳的格式化消息，诊断集群问题时候首先检查此文件
- /var/hacmp/log/hacmp.out，包含脚本执行的每个命令的逐行记录，包括每个命令的所有参数的值
- /var/hacmp/clverify/clverify.log：集群验证详细输出
- /var/hacmp/log/autoverify.log：在自动集群验证运行期间发生的任何警告或错误
- /var/hacmp/log/clutils.log：包含有关日期，时间，结果以及哪个节点执行了自动集群配置验证的信息

在检查启动问题时候，启动HA的时候，跟踪hacmp.out输出查看日志，执行命令：`tail -f hacmp.out`。     
在PowerVM环境，通常要检查netmon.cf配置，路径：/usr/es/sbin/cluster/netmon.cf。

### 日志收集
smit收集cluster log方法：
- 执行命令 ：`smit hacmp`
- 选择选项："Problem Determination Tools"
- 选择选项："PowerHA SystemMirror Log Viewing and Management"
- 选择选项："Collect Log Files for Problem Reporting"
- 各参数说明如下：
    - Log Destination Directory：默认/tmp目录
    - Collection Pass Number：默认为2，1是计算所需空间
    - Nodes to Collect Data from：默认选择all
    - Debug ：默认是No，除非IBM 专家建议
    - Collect RSCT Log Files：默认Yes

命令收集方法：
执行命令`snap -ec`即可，收集数据存放在/tmp/ibmsupt下面，文件名snap.pax.Z。

## 常用命令
PowerHA一般用smit菜单操作，失误率小，偶尔用命令。在AIX系统中，PowerHA命令路径：/usr/es/sbin/cluster。常用命令如下：

命令|功能
:---|:---
clstop |停止集群，注意参数
lscluster -m |查看集群状态
lssrc -g caa |查看caa/rsct/cluster等服务
lssrc -ls cthags |检查组服务子系统
no -a &#124; grep routerevalidate|查看routerevalidate
no -po routerevalidate=1 |修改routerevalidate
ls -al /dev/datavg |查看vg信息，主要是看major号
lvlstmajor |查看可用的 major号
chdev -l fscsi0 -a fc_err_recov=fast_fail -P|修改光纤口属性
chdev -l &#60;disk&#62; -a queue_depth=40 -P|修改磁盘属性示例
/usr/sbin/rsct/bin/dhb_read -p &#60;disk&#62; -r|心跳盘测试(主)
/usr/sbin/rsct/bin/dhb_read -p &#60;disk&#62; -t|心跳盘测试(备)
clctrl -tune -L |查看HA一些参数配置
clctrl -tune -L network_fdt |查看Network network_fdt配置

说明：
- AIX操作系统缓存路由，需要设置routerevalidate选项：`routerevalidate=1`。
- 启停HA命令日常不推荐用，特别是停止，需要加参数指定停止选项。

## 参数配置说明
### 资源组策略说明
"Startup"启动说明：
- Online On Home Node Only:资源组启动期间，仅在其主节点（最高优先级）上启动
- Online On First Available Node:资源组在第一个可用的参与节点上启动
- Online On All Available Nodes:资源组在所有节点上启动
- Online Using Distribution Policy:每个节点上只有一个资源组启动

"Fallover"故障转移：
- Fallover To Next Priority Node in the list:资源组遵循在节点列表中指定的默认节点优先级顺序
- Fallover Using Dynamic Node Priority:节点失败时动态选择迁移节点，一般三节点情况下
- Bring Offline(On Error Node Only:当节点发生error时候，使资源组脱机

"Fallback"退回方式：
- Fallback To Higher Priority Node in the list:当优先级较高的节点加入群集时，资源组将回退
- Never Fallback:资源组加入群集时不会回退到较高优先级的节点

### 资源组节点优先级
对于资源组，节点优先级定义在资源组属性参数"Participating node list"中，说明如下：
- 列表定义了可以承载特定资源组的节点列表，可以是部分节点，也可以是全部
- 默认节点优先级由节点在特定资源组的节点列表中的位置来标识
- 节点列表中的第一个节点具有最高的节点优先级。该节点也称为资源组的主节点。在另一个节点之前列出的节点具有比当前节点更高的节点优先级

### HA停止策略
在停止HA时候，菜单"Stop Cluster Services"中选项"Select an Action on resource groups"
- Bring resource groups Offline:停止正在停止的节点上当前在线的所有受管资源,资源组不会切换到任何节点上
- Move resource groups:停止正在停止的节点上当前在线的所有受管资源,资源组会切换到可用节点上
- Unmanage resource groups:群集服务将立即停止,节点上资源不会停止,应用程序继续运行,PowerHA SystemMirror继续运行，并且RSCT保持运行状态

## 常用操作
### network_fdt参数
&#8195;&#8195;network_fdt即Network Failure Detection Time,在老版本的AIX系统中可能没这个参数，在IV76622中引入了此参数。查看network_fdt参数命令(单位是毫秒)：
```
clctrl -tune -L network_fdt
```
修改network_fdt参数命令(单位是秒，范围5-590)：
```
clmgr modify cluster NETWORK_FAILURE_DETECTION_TIME=<xxx>
```
smit查看及修改方法：
- smit hacmp
- Custom Cluster Configurations
- CLuster Nodes and Networks
- Manage the Cluster
- Cluster heartbeat settings
- Press Enter

官方说明：[PowerHA SystemMirror use of Cluster Aware AIX ](https://www.ibm.com/support/knowledgecenter/zh/SSPHQG_7.2/concept/ha_concepts_ex_cluster.html)

## 常见问题
