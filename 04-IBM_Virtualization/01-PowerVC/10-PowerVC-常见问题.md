# PowerVC-常见问题
## 虚拟分区问题
### 受限制的分区
PowerVC下分发的虚拟机在Profile文件的物理I/O下会有如下提示：
```
Physical I/O
This configuration cannot have physical I/O.
```
查看分区属性，在Advanced Settings中如下选项被勾选：
```
Restricted IO Partition
```
上面选项的勾无法去掉，分区被限制了，I/O只能添加虚拟的。
## 分区配置问题
### VSCSI配置问题
#### 更改默认的VSCSI
&#8195;&#8195;通过PowerVC共享存储池分区的逻辑卷，分区会被自动分配VSCSI，如果想建多个VSCSI分散下磁盘，但是多建的也不会自动均衡分配，此时可以手动进行操作。首先在两个vios下创建vhost，命令示例：
```sh
chhwres -m <Machine name> -p <vios name> -o a -r virtualio \
--rsubtype scsi -s <vios vscsi id> -a \
"adapter_type=server,remote_lpar_name=<LPAR name>,remote_slot_num=<vioc vscsi id>" 
```
然后vioc创建两个vscsi，分别对应两个vios：
```sh
chhwres -m <Machine name> -p <LPAR name> -o a -r virtualio \
--rsubtype scsi -s <vioc vscsi id> -a \
"adapter_type=client,remote_lpar_name=<vios name>,remote_slot_num=<vios vscsi id>" 
```
取消想分配到新建VSCSI上的磁盘，不在PowerVC上操作，直接两个vios上用命令取消：
```sh
rmvdev -vtd <VTD_name>
```
或者HMC上进行操作（V8R7以上版本）：
- HMC中选中需要操作的分区
- 在`Virtual I/O`主选项中选择`Virtual Storage`
- 选择`Shared Storage Pool Volumes`
- 找到需要取消映射的卷，在`Operation`选项中选择`Move`移除映射关系

取消后，进行添加：
- HMC中选中需要操作的分区
- 在`Virtual I/O`主选项中选择`Virtual Storage`
- 选择`Shared Storage Pool Volumes`
- 点击选项`Add Shared Storage Pool Volume`，进入添加对话框：
    - 选择`Storage Cluster`
    - 选择`Add existion Shared Storage Pool Volume`
    - 选择需要添加的卷（每次只能一个，这个设计很low）
    - 选择VIOS的连接，选择两个vios
    - 点击左下角`Edit Connectiong`，弹出编辑对话框：
        - 勾选两个VIOS，指定需要的服务端适配器ID和客户端ID
        - 点击`OK`确认
- 点击`OK`确认

如果报错，并且路径只有单个成功，有个vios失败：
- HMC中选中需要操作的分区
- 在`Virtual I/O`主选项中选择`Virtual Storage`
- 选择`Shared Storage Pool Volumes`
- 找到需要取消映射的卷，在`Operation`选项中选择`Modify Connection`
- 弹出编辑对话框中勾选另一个vios，并指定需要的服务端适配器ID和客户端ID
- 点击`OK`确认

&#8195;&#8195;如果添加成功一个后，添加第一个报错失败，可以刷新下HMC再进行添加。后续每添加一个映射关系都进行刷新下，只能一个一个操作，非常耗费时间。
## 待补充