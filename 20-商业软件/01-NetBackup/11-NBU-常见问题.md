# NBU-常见问题
## 磁盘问题
### 磁盘硬件问题
#### 存储磁盘异常断开
&#8195;&#8195;问题描述：NBU备份使用的后端存储硬件故障，导致所有磁盘离线，NBU软件master节点服务异常，没有发生切换。NBU备份作业完全停止了。master节点报错：
```sh
Fatal error: A disk write to file "/catalog/db/data/NBDB.log" failed with error code: (6)
```
&#8195;&#8195;所在磁盘应该是存储挂载过来的，在存储修复后，master节点可以启动，NBU图形化管理界面能正常登录，手动发起备份，马上就报错，报错示例：
```sh
PM - Error nbim (pid-94579) NBu status: 2106. EMM statys: Storage Server Is down or unavallable
Disk storage server is down (2106)
```
&#8195;&#8195;很明显是后端存储磁盘异常，需要进入Media Server节点查看，此次使用的vxfs，四个节点共享的磁盘，
节点使用`df -h`查看信息如下：
```sh
[root@media1 ~]# df -h
df: /msdp: Input/output error
```
&#8195;&#8195;查看`/etc/fstab`就是需要挂载的vxfs文件系统，无法直接umount，提示busy，当然重新挂载也是不行的，用`vdisk`命令查看查看状态示例如下：
```sh
[root@media1 bin]# vxdisk list -o alldgs
DEVICE          TYPE            DISK         GROUP        STATUS
ibm_ds8x000_200a auto:cdsdisk    nbudg02      nbudg        online dgdisabled
ibm_ds8x000_200b auto:cdsdisk    nbudg03      nbudg        online dgdisabled failing
ibm_ds8x000_200c auto:cdsdisk    nbudg04      nbudg        online dgdisabled
......
media2_disk_0 auto:LVM        -            -            LVM
media2_disk_1 auto:none       -            -            online invalid
```
查看dg状态：
```sh
[root@media1 bin]# vxdg list
NAME         STATE           ID
nbudg        disabled,cds         1573674122.16.media1
```
问题定位明确，后端存储设备恢复后系统磁盘还未恢复，应该是有磁盘锁。第一步停Media Server服务：
```sh
bp.kill_all
```
卸载文件系统：
```sh
umount /msdp
```
`deport Group`操作:
```sh
vxdg deport nbudg
```
操作后`vxdisk list`查看状态示例：
```
[root@media1 bin]# vxdisk list
DEVICE          TYPE            DISK         GROUP        STATUS
ibm_ds8x000_200a auto:cdsdisk    -            (nbudg)      online
ibm_ds8x000_200b auto:cdsdisk    -            (nbudg)      online
......
ibm_ds8x000_2013 auto:cdsdisk    -            (nbudg)      online
media1_disk_0 auto:LVM        -            -            LVM
media1_disk_1 auto:LVM        -            -            LVM
```
`import Group`操作:
```sh
vxdg import nbudg
```
操作后`vxdisk list`查看状态示例：
```
DEVICE          TYPE            DISK         GROUP        STATUS
ibm_ds8x000_201a auto:cdsdisk    nbudg02      nbudg        online
ibm_ds8x000_201b auto:cdsdisk    nbudg03      nbudg        online
ibm_ds8x000_201c auto:cdsdisk    nbudg04      nbudg        online
......
ibm_ds8x000_2026 auto:cdsdisk    nbudg20      nbudg        online
ibm_ds8x000_2027 auto:cdsdisk    nbudg21      nbudg        online
media2_disk_0 auto:LVM        -            -            LVM
media2_disk_1 auto:none       -            -            online invalid
```
挂载文件系统：
```
[root@media2 ~]# mount -a
log replay in progress
replay complete - marking super-block as CLEAN
```
查看挂载：
```
[root@media2 ~]# df -h
Filesystem                Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root      50G  9.9G   41G  20% /
......
/dev/vx/dsk/nbudg/lvmsdp   20T   13T  7.2T  65% /msdp
```
启动服务：
```
[root@media2 ~]# bp.start_all
```
&#8195;&#8195;如果有多个节点，都需要操作一遍。`failing`状态可能还存在没清理掉，但是不影响，可以用命令清理，此次暂时未清理。
## 待补充