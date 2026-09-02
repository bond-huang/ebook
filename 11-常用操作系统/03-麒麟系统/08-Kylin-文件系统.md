# Kylin-文件系统


## 文件系统扩容
### XFS文件系统扩容
在线扩容步骤如下：
- 首先`df -Th`确认要扩容的文件系统类型，此次是XFS类型
- 命令`lsblk`确认新磁盘，例如`/dev/vdc`
- 新磁盘分区：执行`fdisk /dev/vdc`，依次按`n`、`p`、`1`，两次回车用满空间，再按`t`输入`8e`设为LVM类型，最后按`w`保存
- 创建物理卷（PV）：执行`pvcreate /dev/vdc1`，使用命令`pvs`查看已经有pv信息，VG信息无
- 扩展卷组（VG）：用`vgs`查看卷组名（操作系统一般是klas，其它自定义的例如datavg），然后执行`vgextend datavg /dev/vdc1`,使用命令`vgs`查看VG已经扩容
- 扩展逻辑卷（LV）：用`lvs`查看要扩容的LV名（例如/dev/mapper/datavg-datalv），执行`lvextend -l +100%FREE /dev/mapper/datavg-datalv`（将剩余空间全部扩容）
- 扩容文件系统：执行`xfs_growfs /data`(例如/dev/mapper/datavg-datalv挂载点是/data)。注意：XFS必须用`xfs_growfs`命令，不能用`resize2fs`
- 验证结果：执行`df -h`检查确认

注意事项：
- 操作风险：操作前务必备份重要数据，尽量在业务低峰期进行
- 禁止缩容：XFS文件系统不支持缩容，操作前务必确认容量无误
- XFS必须用`xfs_growfs`命令，不能用`resize2fs`，使用`resize2fs`文件系统会损坏

## 待补充