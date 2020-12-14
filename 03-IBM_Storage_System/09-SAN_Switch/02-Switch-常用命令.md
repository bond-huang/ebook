# Switch-常用命令
交换机管理常用命令
## SAN B-type 常用命令
IBM SAN B-type系列交换机是OEM博科的，命令和博科交换机基本一致。
示例：
```shell
configshow -pattern "fabric.ops"
```
### 基本信息查看
&#8195;&#8195;对于高端B384这种交换机，&#60;port_index&#62;有些命令不是index，而是slot加port，例如index是84，solt号是2，port号是5，那么就是2/5。常用命令如下：

命令|用途
---|:---
chassisshow|查看交换机chassis信息
switchshow|查看交换机配置信息
ipaddrshow|查看交换机ip地址
switchStatusshow|查看交换机健康状态
portshow &#60;port_index&#62;|查看某个端口状态，NPIV的虚拟WWN查看
slotshow|查看交换机slot状态
sfpshow &#60;port_index&#62;|查看某个sfp状态
porterrshow|查看端口数据和数据类型错误统计信息
pshow|查看交换机电源状态
fanhow|查看交换机风扇状态
licenseshow|查看交换机license信息
firmwareshow|查看交换机微码信息
hashow|查看交换机ha状态
tempshow|查看交换机环境温度
islshow|查看thunk连接状态
errdump|查看交换机日志
supportshow|查看交换机诊断信息
fabricShow|显示fabric信息及级联信息
nsshow|查看当前交换机设备信息
nsallshow|查看fabric中的所有设备信息

### zone配置相关
创建一个zone的基本步骤：
- 创建alias：`aliCreate`
- 创建zone：`zoneCreate`
- 创建cfg或者将zone添加到现有cfg：`cfgCreate` or `cfgAdd`
- 保存配置：`cfgSave`
- 激活配置：`cfgEnable`

zone配置相关常用命令如下：

命令|用途
---|:---
aliCreate|创建别名
aliAdd|在别名中添加成员
aliRemove|删除别名成员
aliDelete|删除别名
aliShow|查看别名信息
zoneCreate|创建zone
zoneAdd|zone中添加成员
zoneMove|zone中删除成员
zoneDelete|删除zone
zoneShow|查看zone配置
cgfCreate|创建zone配置文件
cfgAdd|指定zone配置文件中添加zone
cfgRemove|指定zone配置文件中删除zone
cfgDelete|删除zone配置文件
cfgShow|查看所有zone配置信息
cfgSave|保存zone配置信息
cfgTransAbort|撤销所有上一次cfgSave之后未保存的更改
cfgEnable|激活配置
cfgDisable|disable zone配置
configUpload|备份配置
configDownload|恢复配置

以wwnzone为示例,使用格式如下：
```shell
aliCreate "DE4000H_A_port2","20:22:d0:39:ea:1a:de:71"
aliCreate "DE4000H_B_port2","20:22:d0:39:ea:1a:9d:61"
aliCreate "vios1_fcs1","10:00:00:10:9b:66:ac:1e"
aliCreate "vios2_fcs1","10:00:00:10:9b:66:da:b7"
zoneCreate "DE4000H_vios1","DE4000H_A_port2;DE4000H_B_port2;vios1_fcs1;vios2_fcs1"
zoneCreate "DE4000H_vios2","DE4000H_A_port2;DE4000H_B_port2;vios1_fcs1;vios2_fcs1"
cfgAdd "cfg_A","DE4000H_vios1;DE4000H_vios2"
cfgEnable "cfg_A"
cfgSave
```
注意：      
&#8195;&#8195;建议先cfgEnable然后cfgSave。先cfgSave的话，会有提示defind和effective的配置会不一致，并且在错误日志里面有warning：defind and effective zone configurations are inconsistent。

### 修改配置
常用对交换机配置修改命令如下：

命令|用途
---|:---
portdisable|disable一个端口
portenable|enable一个端口
portstatsclear|清除交换机指定端口状态
statsclear|清除端口诊断统计信息
chassisdisable|disable chassis上所有端口
chassidenable|enbale chassis上所有端口
swichdisable|disable 交换机
switchenable|enbale 交换机
passwd|修改用户密码
errclear|清除错误日志
hafailover|切换HA
ipaddrset|IP地址配置
configuration|配置菜单，例如配置Domain ID

## SAN C-type 常用命令
暂未接触
