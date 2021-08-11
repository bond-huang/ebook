# AIX-NIMserver
NIMserver功能很强大，目前用的比较多的就是mksys备份恢复系统及升级系统等。
## AIX系统备份
### 常规备份注意
mksysb的前提条件：
- 备份的目录有足够的空间
- 备份时确保系统活动减少到最低限度

&#8195;&#8195;如果需要排除某个文件系统或目录，编辑/etc/exclude.rootvg文件。例如，要排除名为`temp`的目录的所有内容，格式示例：
```
/temp/
```
要排除名为`/tmp`的目录的内容，并避免排除路径名中带有`/tmp`的任何其他目录，格式示例：
```
^./tmp/
^./backup/
```
排除exclude.rootvg文件中目录的备份操作（其它默认）：
- smit mksysb
- Enter backup device or file
- Change "EXCLUDE files?" form "no" to "yes"

对应上面操作的命令是：
```
mksysb -e -i -A /tmp/mksysb0312.sysb
```
参数解释：
- 参数`-e`：在备份时排除/etc/exclude.rootvg文件中列示的文件
- 参数`-i`：调用生产/image.data文件的`mkszfile`命令
- 参数`-A`: 备份DMAPI文件系统文件

### 不带镜像的备份
&#8195;&#8195;在AIX系统中，对rootvg进行备份使用mksysb时候，默认会在根目录下创建image.data文件，mksysb会根据此文件生成对应的镜像。如果原系统有镜像两块磁盘，直接做mksyb，恢复分区只有一块磁盘，在恢复时候会报错，提示空间不够，有几种解决方法：
- 解除原分区的镜像，做mksyb后再重新做镜像（对一些高要求系统来说存在风险）
- 搞一块新的临时硬盘到新分区（也许配置会比较麻烦，后续可能会删掉概率也大）
- 修改image.data文件，在做mksysb的时候不创建新的image.data，使用修改的image.data

IBM 官方介绍：[Creating and Restoring a Mksysb Without Preserving Mirrors](https://www.ibm.com/support/pages/creating-and-restoring-mksysb-without-preserving-mirrors-using-nim-tape-or-disk)

本人写了一个脚本修改image.data文件，参考链接：[Shell-mksysb相关脚本](https://ebook.big1000.com/09-Shell%E8%84%9A%E6%9C%AC/03-Shell_AIX%E8%84%9A%E6%9C%AC/02-Shell-mksysb%E7%9B%B8%E5%85%B3%E8%84%9A%E6%9C%AC.html)

新建image.data文件：
```
mkszfile
```
文件示例：
```
lv_data:
        VOLUME_GROUP= rootvg
        LV_SOURCE_DISK_LIST= hdisk0 
        LV_IDENTIFIER= 00cb4d6e00004c0000000168788002ed.1
        LOGICAL_VOLUME= hd5
        VG_STAT= active/complete
        TYPE= boot
        MAX_LPS= 512
        COPIES= 1
        LPs= 1
        STALE_PPs= 0
        INTER_POLICY= minimum
        INTRA_POLICY= edge
        MOUNT_POINT= 
        MIRROR_WRITE_CONSISTENCY= on/ACTIVE
        LV_SEPARATE_PV= yes
        PERMISSION= read/write
        LV_STATE= closed/syncd
        WRITE_VERIFY= off
        PP_SIZE= 128
        SCHED_POLICY= parallel
        PP= 1
        BB_POLICY= non-relocatable
        RELOCATABLE= no
        UPPER_BOUND= 32
        LABEL= primary_bootlv
        MAPFILE=
        LV_MIN_LPS= 1
        STRIPE_WIDTH= 
        STRIPE_SIZE= 
        SERIALIZE_IO= no
        FS_TAG=  
        DEV_SUBTYP=
```
需要修改的项目：
- LV_SOURCE_DISK_LIST= hdisk0:如果两块盘互为镜像这里会显示两个盘符，不需要镜像保留一个即可
- COPIES= 1：不需要镜像就改成1
- PP= 20：根据LV的情况来判断，和`LPs`的值保持一致

使用vi修改命令示例：
```
:%s/COPIES= 2/COPIES= 1/g
```
## AIX系统恢复
&#8195;&#8195;AIX系统从mksysb恢复通常使用NIMserver比较多，也可以把mksysb做成一个启动CD，刻录出来或者通过虚拟光驱去安装。恢复过程中注意事项：
- 如果对端分区没开起来，NIM分发系统时候会报网络问题：code78，可以忽略
- 在分发过程中好像不能再开个shell再次分发，但是在分发完成后系统安装过程中可以再分发系统
- 确保`/etc/hosts`表里面IP和hostname都是一一对应关系，如果一个IP对应不同hostname，系统安装完成后hostname可能不是预期的

系统迁移通过mksysb恢复的系统需要注意事项：
- hostname名不能变，可能导致数据库启动不了
- /etc/hosts表必须一致
- 心跳vg的磁盘的hdiskX编号必须一致，网卡的编号可以不一致
- HA 6.1 /usr/es/sbin/cluster/etc/rhosts 需配置
- HA 7.1 etc/rhosts需配置一致
- PowerVM 环境下/usr/es/sbin/cluster/netmon.cf需配置
- 数据库裸设备可能需要更改权限
- AIX 7.1心跳vg需要重新配置

## NIM相关操作
### resetmachine
有时候删除定义machine时候需要先reset一下：
- smit nim
- Perfom NIM Administration Tasks
- Manage Machines
- Perform Operations on Machines
- 选择需要reset的machines进行reset
- 在reset the NIM State of a machie菜单回车即可reset

如果是想删除machines定义，在reset the NIM State of a machie菜单中将两个选项都改成`yes`。

## mksysb资源定义并分发 

## AIX安装镜像定义并分发


## 常见问题
### host表IP冲突
当host表里面有一个IPA对应hostA,如果在定义一个host，IPA对应hostB，定义B名字的machine后，分发系统会报错，即使后期删掉了IPA对应主机名A的关系，一样会报错。提示：
```
rc=0
exportfs: hostA:unkonwn host
warning:warning:0042-006 m_bos_inst:(From_Master) connnetc Connection timed out
```
