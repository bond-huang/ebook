# SVC-远程拷贝
SVC远程拷贝(Remote Copy)功能是SVC一项重要功能，可用于数据冗余、数据备份、数据迁移及容灾数据同步等。

### 拷贝模式
SVC提供了三种一致性拷贝模式：
- 高速镜像：采用数据实时同步方式
- 全局镜像：此模式是异步模式，在写操作应用到主卷后再将其应用到辅助卷，这需要两个站点直接的高带宽链接，以防止在发生故障时丢失数据
- 具有变更卷的全局镜像：此方式也是异步模式。在使用此方式时候，将跟踪变更，并将变更拷贝到中间变更卷，变更将定期传输至辅助站点以降低带宽需求（时间可以设置从60s至86400s）

在SVC Management GUI中，根据图标就可以看出同步采用的是哪种类型，如图所示：

![拷贝模式](拷贝模式.png)

### 高速镜像修改成全局镜像（异步）
例如一个一致性组test，状态是一致同步状态，采用高速镜像（同步）模式，修改成全局镜像（异步）步骤如下：
- 选择一致性组test，右键点击“停止”选项
- 对话框提示是否允许辅助卷的读写访问，不勾选“允许辅助读/写访问”（看配置需求）
- 然后点击“停止一致性组”，可以看到同步关系是“一致停止”
- 选择一致性组test，右键点击“编辑一致性组”选项
- 对话框“编辑一致性组”中类型选择“全局镜像”，循环方式“无”，循环周期默认“300秒”
- 点击确认下一个对话框会提示：该关系中的卷是否已同步，选择“否，卷未同步”
- 选择一致性组test，右键点击“启动”选项
- 状态为“不一致拷贝”，可以任务列表里面查看拷贝进度
 
### 高速镜像修改成具有变更卷的全局镜像（异步）
例如一个一致性组test，状态是一致同步状态，采用高速镜像（同步）模式，修改具有变更卷的全局镜像（异步）步骤如下：
- 前面步骤同上1-4步骤；
- 对话框“编辑一致性组”中类型选择“全局镜像”，循环方式选择“多个”，循环周期默认“300秒”，循环周期根据需求定义在60s至86400s之间
- 在修改完成后，启动一致性组，是无法启动的，提示没有主变更卷
- 不需要删除一致性关系，选中需要一致性组中的一致性关系，右键选择“全局镜像变更卷”
- 可以选择创建新的或者添加新的卷（此卷必须与关系中的其它卷在同一个I/O组，并且具有相同的容量，无法将改卷映射给主机，或者在其它拷贝服务中使用）。
- 同样在辅助SVC端也要创建同样的辅助全局变更卷，才可以启动一致性组。

### 一致性组相关命令
有时候需要用到一些命令去查看或者编辑一致性组关系，用的很少，简单记录几个。
##### 查看一致性组
查看所有及指定某一个查看的命令，不知道ID先查看所有：
```
lsrcconsistgrp -delim :
lsrcconsistgrp -delim : rc_consist_group_id
```
说明：rc_consist_group_id是一致性组的ID
##### 创建一致性组
创建一致性组及往一致性组里面添加关系：
```
mkrcconsistgrp -name new_name -cluster cluster_id
chrcrelationship -consistgrp consist_group_name rc_rel_id 
```
说明：
- new_name是新一致性组的名称，cluster_id是新一致性组 的远程集群的ID
- 如果 未指定-cluster，则仅在本地群集上创建一致性组- 
- 新的一致性组不包含任何关系，并且处于空状态。
- consolid_group_name是要为其分配关系的新一致性组的名称，而rc_rel_id是该关系的ID

##### 启动及停止一致性组
启动及停止一致性组以及用`-access`参数启用辅助卷的写访问：
```
startrcconsistgrp rc_consist_group_id
stoprcconsistgrp rc_consist_group_id 
stoprcconsistgrp rc_consist_group_id -access 
```
说明：rc_consist_group_id是一致性组的ID
##### 删除一致性组
删除一致性组（里面没有一致性关系），以及强制删除（不为空，里面还有一致性关系）：
```
rmrcconsistgrp rc_consist_group_id 
rmrcconsistgrp -force rc_consist_group_id 
```
说明：rc_consist_group_id是一致性组的ID
##### 更改一致性组
全局镜像更改为高速镜像：
```
chrcconsistgrp  -metro rc_consist_group_name
```
高速镜像更改为全局镜像：
```
chrcconsistgrp -global -cyclingmode none rc_consist_group_name
```
高速镜像更改为具有变更卷的全局镜像：
```
chrcconsistgrp -global -cyclingmode multi -cycleperiodseconds period rc_consist_group_name
```
说明：
- rc_consist_group_name是要更改的一致性组的名称
- 更改后所有新设置都适用于一致性组内的所有关系
