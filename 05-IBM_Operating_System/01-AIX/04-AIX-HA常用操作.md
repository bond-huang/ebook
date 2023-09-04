# AIX-HA常用操作
PowerHA是AIX系统中常用的高可用软件。
## 日志查看和收集
### 日志查看
大多数HA日志存放在/var/hacmp目录下，常用的有：
- 命令`errpt -e`输出：系统日志，会有HA集群事件，也可以查看/var/adm/ras/errlog文件
- /var/hacmp/adm/cluster.log：包含由PowerHA SystemMirror脚本和守护程序生成的带时间戳的格式化消息，诊断集群问题时候首先检查此文件
- /var/hacmp/log/hacmp.out，包含脚本执行的每个命令的逐行记录，包括每个命令的所有参数的值
- /var/hacmp/clverify/clverify.log：集群验证详细输出
- /var/hacmp/log/autoverify.log：在自动集群验证运行期间发生的任何警告或错误
- /var/hacmp/log/clutils.log：包含有关日期，时间，结果以及哪个节点执行了自动集群配置验证的信息
- /var/adm/ras/syslog.caa:CAA日志，CAA即Cluster-Aware AIX

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

## 心跳盘操作
### 更改心跳盘
参考链接：[Replacing a repository disk with SMIT](https://www.ibm.com/docs/en/powerha-aix/7.2?topic=networks-replacing-repository-disk)
## 磁盘操作
### AIX系统层面磁盘迁移
参考链接：[13 steps to migrate PowerHA shared volume groups to new storage while keeping a copy on the old disks](https://www.ibm.com/support/pages/13-steps-migrate-powerha-shared-volume-groups-new-storage-while-keeping-copy-old-disks#:~:text=Add%20the%20new%20disks%20to%20the%20shared%20volume,confirm%20adding%20the%20disk%20to%20the%20volume%20group.)
### 非并发VG添加到HA
参考链接：[How to add an existing nonconcurrent volume group to PowerHA resource group](https://www.ibm.com/support/pages/how-add-existing-nonconcurrent-volume-group-powerha-resource-group)
## 常见问题
### 同步问题
问题一：    
&#8195;&#8195;同步时候报error时间戳问题，在同步时候选项中第二个选项选择yes可以自动修复这个问题（在HA停止状态下才有此选项），
Force synchronization if verification fails? [No] to 
Force synchronization if verification fails? [Yes] 

问题二：    
&#8195;&#8195;A是主节点在运行资源组，B是备节点未启动，A可以同步到B（可能有warring说配置不一致），B也可以启动，但是启动后查看状态B看到A是down，A看到B是down，如果尝试从A向B切换，会发现选择不了B节点的，建议步骤：停掉B备节点，然后检查两个节点的rhost文件：
- HA6:etc/es/sbin/cluster/rhosts
- HA7:/etc/cluster/rhosts

### 切换问题
问题一：      
&#8195;&#8195;在切换HA过程中很慢，看日志是发现HA脚本出现问题过不去，执行选项
`Recover From PowerHA SystemMirror Script Failure`即可，然后去检查修复脚本的问题。        
问题二:     
&#8195;&#8195;A是主节点在运行资源组，B是备节点HA也在运行，状态看都正常，但是从A向B切换后发现部分文件系统挂载，然后又自动卸载，查看状态节点资源组是error状态，如果排除脚本的问题，那可能是文件系统配置问题，建议检查/etc/filesystems是否一致，还有就是检查文件系统，是否有在B机单独在HA的文件系统中加过文件或者目录或者软链接等。       
问题三:       
&#8195;&#8195;A往B切换过程中出现问题，A变成error状态，资源组两边都没有，执行选项
`Recover From PowerHA SystemMirror Script Failure`后HA状态看起来正常，但是B机的NAS文件系统未挂载（在HA脚本中有写挂载），手动挂载后，发现A机`df -g`命令有报错，并且不会自动返回到shell。检查解决步骤：
- 检查rc.local和inittab都没有异常的选项
- 检查A机HA停止脚本中有无效的NFS文件系统umonut选项，注释或者删除
- 检查A机HA停止脚本中有漏掉的NFS文件系统umonut选项，添加上去
- 手动执行umonut漏掉的NFS文件系统
- 执行后发现有操作系统的文件系统umount了，手动mount或者重启即可

### 节点通信问题
&#8195;&#8195;通常情况下，节点通信问题原因可能有：网络问题，配置问题等等，配置例如host表，rhost文件，IP配置，wlan配置问题等等，但是有时候问题不是很明显，最近遇到过一个HA配置无问题，网络看起来也是正常的，但是HA同步就同步不了。   
例如：A节点HA未启动，B节点HA启动了，资源组正在运行，AB节点都是PowerVM环境的VIOC。   
从B往A同步执行等半天后，发现报错如下：
```
ERROR:Comm error found on node:<nodename>
```
&#8195;&#8195;在B节点Show Cluster Services菜单中，执行后很慢出结果，等待结果出来后发现A节点Cluster-Aware AIX status是down的状态，在A节点上检查服务状态：
```
# lssrc -a |grep inetd
 inetd            tcpip            2490704      active
# lssrc -a |grep cthags
 cthags           cthags                        inoperative
# lssrc -a |grep clcomd
 clcomd           caa              6750706      active
# lssrc -a |grep clconfd
 clconfd          caa              12911042     active
# lssrc -ls inetd
Subsystem         Group            PID          Status
 inetd            tcpip            2490704      active
Debug         Not active
Signal        Purpose
 SIGALRM      Establishes socket connections for failed services.
 SIGHUP       Rereads the configuration database and reconfigures services.
 SIGCHLD      Restarts the service in case the service ends abnormally.
Service       Command                  Description              Status
 caa_cfg      /usr/sbin/clusterconf    clusterconf              active
 ftp          /usr/sbin/ftpd           ftpd                     active
```
从B向A或A向B ping都看起来正常：
```
$ ping -S nodea nodeb
PING nodeb: (1.1.253.130): 56 data bytes
64 bytes from 1.1.253.130: icmp_seq=0 ttl=255 time=0 ms
64 bytes from 1.1.253.130: icmp_seq=1 ttl=255 time=0 ms
64 bytes from 1.1.253.130: icmp_seq=2 ttl=255 time=0 ms
```
使用`lscluster -m`命令去查看集群状态，可以看到A节点下有如下显示：
```
Points of contact for node :0
```
在A节点的HA Show Repository Disks菜单中执行后获取不到心跳盘的部分信息，并会有报错（时有时无）：
```
ERROR:unable to communicate with "nodeb"(nodeb,ip)
ERROR:"Repository Disks PVID" was not found in data that you requested.
```
在B节点的HA Show Repository Disks菜单可以获取到心跳盘信息，有时也会有：
```
Communication error, see logfile
```
以上菜单都执行比较慢，但是使用dhb_read命令去测试两个节点间心跳磁盘，测试是正常的。

lspv查看caavg_private VG的状态是active，检查caa日志，会发现如下报错：
```
tcpsock_ndd_add2_sendq: Overflow, Dropping packet.
Could not send lock request message. rc=127
cluster_utils.c get_cluster_lock...Could not get lock....
```
&#8195;&#8195;可能是在节点A上启用了jumbo frames,而B节点无法接收jumbo frames，可以分别在两个node上执行如下命令，如果Jumbo frame启用了，那么MTU size = 9000：
```
# netstat -i
Name   Mtu   Network     Address                 Ipkts     Ierrs        Opkts     Oerrs  Coll
en1    1500  link#2      ee.37.5f.fc.eb.3        281954049     0         33827048     0     0
en1    1500  10.8.253    npospdb2-cv             281954049     0         33827048     0     0
en0    1500  link#3      ee.37.5f.fc.eb.2        901593154     0        586172871     0     0
en0    1500  192.168.3   node_svc                901593154     0        586172871     0     0
en0    1500  1.1.253     nodeb                   901593154     0        586172871     0     0
```
&#8195;&#8195;此次没有启动jumbo frames，但也是是网络通信问题，检查HA boot ip所在网卡的属性，如果largesend是开启的，在这种情况下，VIOS上的相应适配器也应启用largesend和large_receive，以避免在管理程序中出现碎片。可以使用命令clrsh测试节点连接：
```
# clrsh nodea date
Communication error, see logfile
# clrsh nodeb date
Thu Feb 18 11:07:15 CST 2021
# hostname
nodea
# clrsh nodea date
Thu Feb 18 11:08:24 CST 2021
# clrsh nodeb date
Thu Feb 18 11:08:34 CST 2021
# clrsh nodea date
Thu Feb 18 11:08:46 CST 2021
# clrsh nodeb date
Communication error, see logfile
```
可以看到有时候通信异常，有时候能出现命令输出也比较慢。

解决方法可以禁用mtu_bypass：
```
# chdev -l enX -a mtu_bypass = off
```
或在VEA或中继VEA上禁用PLSO：
```
# chdev -l entX -a platform_lso = no 
```
IBM相关问题描述链接：
- [IJ25390: SLOW TCP PERFORMANCE WITH VIRTUAL ETHERNET ADAPTER AND LARGESENDAPPLIES TO AIX 7200-04](https://www.ibm.com/support/pages/apar/IJ25390)
- [VIOS (Doc Number=6667): High Impact / Highly Pervasive APAR IJ25390](https://www.ibm.com/support/pages/node/6231440)

### 磁盘问题
#### PVID问题
&#8195;&#8195;有时候由于某些异常原因导致某一节点的数据磁盘的PVID丢失或者不一致，例如备节点test1磁盘PVID异常，同步时候报错如下：
```
ERROR: A disk with PVID 000baf2b911918a40000000000000000 is a part of the volume group datavg participating in resource group test_rg on node test2. Node test1 is also
part of this resource group, but it does not have this PVID defined as a
part of this volume group.
The list of PVIDs per volume group should be consistent for all nodes that
can require access to this volume group.
Starting Corrective Action: cl_resource_update_vg_definitions.
Do you want to change volume group definitions for the volume
group: datavg participating in resource group test_rg on node: test1 [Yes / No]:
<01> Updating volume group definitions of shared VG: datavg participating in 
resource group test_rg so that it will be consistent across all the nodes 
from this resource group: FAIL   
```
如果磁盘`reserve_policy`属性是`single_path`则改成`no_reserve`：
- 如果PVID不一致，修改PVID为一致，如果磁盘`reserve_policy`属性是`single_path`则改成`no_reserve`,  然后同步，备机test1的VG信息恢复，HA启动正常
- 如果test1的disk PVID为`NONE`，使用命令`odmdelete -o CuDv -q "name=hdisk3"`删除ODM信息，然后重启操作系统，磁盘PVID和VG信息应该会恢复

修改PVID方法(每两位从十六进制转换成八进制)，示例：
```sh
echo "\0000\0013\0257\0053\0231\0030\0030\0244\c" > /tmp/myPVID
cat /tmp/myPVID |dd of=/dev/hdisk3 bs=1 seek=128
lquerypv -h /dev/hdisk3 80
rmdev -dl hdisk3 
cfgmgr
lspv
```
或者使用写的Python脚本修改：[Python-AIX配置修改-自动修改hdisk的PVID](http://ebook.big1000.com/08-Python/04-Python_AIX%E8%84%9A%E6%9C%AC/03-Python-AIX%E9%85%8D%E7%BD%AE%E4%BF%AE%E6%94%B9.html)
#### 磁盘异常丢失
&#8195;&#8195;模拟磁盘丢失，断掉备机test1分区的hdisk3磁盘映射，双vios都断开，磁盘开始报路径丢失和磁盘操作错误，但是VG状态不变，磁盘状态看起来正常，主机端test2没有errpt：
```
test1:/#lspv
hdisk0          00f939e23e8ab01a                    rootvg          active      
hdisk1          000baf2b911815a4                    datavg          concurrent  
hdisk2          000baf2b9118164a                    caavg_private   active 
hdisk3          000baf2b9e10273d                    datavg          concurrent      
```
在主机test2新建文件系统，备机test1的`datavg`状态从`concurrent`变空，并且主机test2报错：
```
test2:/#errpt
IDENTIFIER TIMESTAMP  T C RESOURCE_NAME  DESCRIPTION
F7DDA124   1209160521 U H LVDD           PHYSICAL VOLUME DECLARED MISSING
```
备机test1报错：
```test1:/#errpt
IDENTIFIER TIMESTAMP  T C RESOURCE_NAME  DESCRIPTION
AEA055D0   1209160521 I S livedump       Live dump complete
CAD234BE   1209160521 U H LVDD           QUORUM LOST, VOLUME GROUP CLOSING
52715FA5   1209160521 U H LVDD           FAILED TO WRITE VOLUME GROUP STATUS AREA
F7DDA124   1209160521 U H LVDD           PHYSICAL VOLUME DECLARED MISSING
E86653C3   1209160521 P H LVDD           I/O ERROR DETECTED BY LVM
B6267342   1209160521 P H hdisk3         DISK OPERATION ERROR
```
命令`rmdev -Rdl hdisk3`删除不了磁盘，直接`odmdelete -o CuDv -q "name=hdisk3"`删除ODM库，可以删除。然后VIOS重新映射磁盘，报错：
```
"hdisk7" is already being used as a backing device.  Specify the -f flag
to force this device to be used anyway.
```
强制映射：
```
mkvdev -vdev hdisk7 -vadapter vhost3 -dev vtscsi8 -f
```
&#8195;&#8195;磁盘使用命令`cfgmgr`扫描不回来，停掉备机test1的HA，重启系统，重启后hdisk3恢复，并且vg信息也有。再次用命令`rmdev -Rdl hdisk3`删掉磁盘，`cfgmgr`扫描回来后依然正常。

### 宕机问题
宕机原因参考链接：
- [TWT-如何处理hacmp中dms的问题](https://www.talkwithtrend.com/Article/7729)
- [Toolbox-LPAR AIX Rebooted Auto](https://www.toolbox.com/tech/operating-systems/question/lpar-aix-rebooted-auto-020915/)
- [DB2-Diagnosing a host reboot with a restart light](https://www.ibm.com/docs/en/db2/10.1.0?topic=host-diagnosing-reboot-restart-light)
- [GPFS filesystem outage with a kernel panic](https://www.ibm.com/docs/en/db2/10.5?topic=light-gpfs-filesystem-outage-kernel-panic)
- [IBM Spectrum Scale filesystem outage with a kernel panic](https://www.ibm.com/docs/en/db2/11.1?topic=light-spectrum-scale-filesystem-outage-kernel-panic)
- [IV87544: SHUTDOWN -F ON POWERHA MAY PANIC INSTEAD OF HALT](https://www.ibm.com/support/pages/apar/IV87544?mhsrc=ibmsearch_a&mhq=APAR%20IV69760%20)
- [IV69760: NODE DOWN IN CAA CLUSTER DUE TO CONFIGRM MEMORY LEAK](https://www.ibm.com/support/pages/apar/IV69760?mhsrc=ibmsearch_a&mhq=APAR%20IV69760%20)

PowerHA集群与宕机相关原理等：
- [RSCT3.2-Action 9: investigate an AIX node crash](https://www.ibm.com/docs/en/rsct/3.2?topic=recoveries-action-investigate-aix-node-crash)
- [Adjusting Dead Man Switch Timeout for CLUSTER (DMS)](https://www.ibm.com/support/pages/node/519603?mhsrc=ibmsearch_a&mhq=KERNEL_PANIC%20IDENTIFIER%3A%20%20%20%20%20225E3B63)

### 待补充