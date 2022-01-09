# RedHat-磁盘管理
使用RedHat8系统学习磁盘管理时候记录的学习笔记。
## 磁盘空间管理
### 磁盘空间使用情况
使用df命令查看文件系统空间使用情况，练习示例：
```
[root@redhat8 linuxone]# df 
Filesystem            1K-blocks    Used Available Use% Mounted on
devtmpfs                 909368       0    909368   0% /dev
tmpfs                    924716       0    924716   0% /dev/shm
tmpfs                    924716    9548    915168   2% /run
tmpfs                    924716       0    924716   0% /sys/fs/cgroup
/dev/mapper/rhel-root  17811456 4183628  13627828  24% /
/dev/nvme0n1p1          1038336  172892    865444  17% /boot
tmpfs                    184940      16    184924   1% /run/user/42
tmpfs                    184940       4    184936   1% /run/user/1000
[root@redhat8 linuxone]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
tmpfs                  904M     0  904M   0% /dev/shm
tmpfs                  904M  9.4M  894M   2% /run
tmpfs                  904M     0  904M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  4.0G   13G  24% /
/dev/nvme0n1p1        1014M  169M  846M  17% /boot
tmpfs                  181M   16K  181M   1% /run/user/42
tmpfs                  181M  4.0K  181M   1% /run/user/1000
```
详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-df.html](https://www.runoob.com/linux/linux-comm-df.html)
### 磁盘空间占用情况
使用du命令查看指定路径下文件占用情况：
```
[root@redhat8 usr]# cd /var/log
[root@redhat8 log]# du
...
3092	./audit
108	./rhsm
64	./tuned
4052	./anaconda
0	./httpd
12544	.
[root@redhat8 log]# du -s
12544	.
[root@redhat8 log]# du -sm
13	.
[root@redhat8 log]# du -sh
13M	.
[root@redhat8 /]# du -s /var/log
12544	/var/log
[root@redhat8 /]# du -sh /var/log
13M	/var/log
```
详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-du.html](https://www.runoob.com/linux/linux-comm-du.html)
## 系统挂载命令
### mount命令
mount命令详细介绍及学习参考链接：[https://www.runoob.com/linux/linux-comm-mount.html](https://www.runoob.com/linux/linux-comm-mount.html)

显示当前系统挂载情况：
```
[root@redhat8 /]# mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=909368k,nr_inodes=227342,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
...
```
`/etc/fstab`是系统挂载定义文件，每行为一个挂载定义，格式示例如下：
```
[root@redhat8 /]# tail -n 5 /etc/fstab
/dev/mapper/rhel-root   /                       xfs     defaults        0 0
UUID=0a510af6-8abf-4e64-b559-902804c93568 /boot                   xfs     defaults        
0 0/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
/tmp/not-existing	/mnt/not-existing	iso9660 default		0 0
/tmp/test		/mnt/test		iso9660 default		0 0
```
说明：
- mount命令使用只指定源设备路径或者挂载点时候，会查询`/etc/fstab`文件，有相同定义则执行挂载
- 在`/etc/fstab`文件的挂载定义中`noauto`是个特殊参数，表示系统启动时不进行自动挂载
- `mount -a`命令会根据当前`/etc/fstab`文件执行挂载操作；每次修改`/etc/fstab`文件后必须执行此命令确认，有错误回显要及时处理，如未处理将会导致系统启动失败

使用mount命令挂载光盘示例：
```
[root@redhat8 /]# mount /dev/cdrom /mnt
mount: /mnt: WARNING: device write-protected, mounted read-only.
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/sr0               6.7G  6.7G     0 100% /mnt
```
修改`/etc/fstab`文件挂载光盘：
```
[root@redhat8 /]# tail -n 1 /etc/fstab
/dev/sr0		/mnt			iso9660	noauto		0 0	
[root@redhat8 /]# tail -n 1 /etc/fstab
/dev/sr0		/mnt			iso9660	noauto		0 0	
[root@redhat8 /]# mount /dev/sr0
mount: /mnt: WARNING: device write-protected, mounted read-only.
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/sr0               6.7G  6.7G     0 100% /mnt
[root@redhat8 /]# umount /mnt
[root@redhat8 /]# mount /mnt
mount: /mnt: WARNING: device write-protected, mounted read-only.
```
在`/etc/fstab`文件中写入无效的定义，使用`mount -a`命令：
```
[root@redhat8 /]# tail -n 5 /etc/fstab
UUID=0a510af6-8abf-4e64-b559-902804c93568 /boot                   xfs     defaults        
0 0/dev/mapper/rhel-swap   swap                    swap    defaults        0 0
/tmp/not-existing	/mnt/not-existing	iso9660 default		0 0
/tmp/test		/mnt/test		iso9660 default		0 0
/dev/sr0		/mnt			iso9660	noauto		0 0	
[root@redhat8 /]# mount -a
mount: /mnt/not-existing: mount point does not exist.
mount: /mnt/test: mount point does not exist.
```
### umount命令
umount命令详细介绍及学习参考链接:[https://www.runoob.com/linux/linux-comm-umount.html](https://www.runoob.com/linux/linux-comm-umount.html)

使用示例：
```
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/sr0               6.7G  6.7G     0 100% /mnt
[root@redhat8 /]# umount /dev/sr0
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
[root@redhat8 /]# mount /mnt
mount: /mnt: WARNING: device write-protected, mounted read-only.
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/sr0               6.7G  6.7G     0 100% /mnt
[root@redhat8 /]# umount /mnt
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
```
## 磁盘分区管理
### fdisk命令
查看现有磁盘分区：
```
[root@redhat8 /]# fdisk -l
Disk /dev/nvme0n1: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8b66746c
...
```
添加一个3G大小的磁盘：
```
[root@redhat8 /]# fdisk -l
...
Disk /dev/nvme0n2: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
...
```
在磁盘`/dev/nvme0n2`上创建新的两个primary分区，一个extended分区：
```
[root@redhat8 ~]# fdisk /dev/nvme0n2
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x1670baa7.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 
First sector (2048-6291455, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-6291455, default 6291455): +1G  
Created a new partition 1 of type 'Linux' and of size 1 GiB.

Command (m for help): n  
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 
First sector (2099200-6291455, default 2099200): 
Last sector, +sectors or +size{K,M,G,T,P} (2099200-6291455, default 6291455): +1G
Created a new partition 2 of type 'Linux' and of size 1 GiB.

Command (m for help): n 
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (3,4, default 3): 
First sector (4196352-6291455, default 4196352): 
Last sector, +sectors or +size{K,M,G,T,P} (4196352-6291455, default 6291455): 
Created a new partition 3 of type 'Extended' and of size 1023 MiB.

Command (m for help): wq
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
查看新建的分区：
```
[root@redhat8 ~]# fdisk -l
...
Disk /dev/nvme0n2: 3 GiB, 3221225472 bytes, 6291456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9711dc11

Device         Boot   Start     End Sectors  Size Id Type
/dev/nvme0n2p1         2048 2099199 2097152    1G 83 Linux
/dev/nvme0n2p2      2099200 4196351 2097152    1G 83 Linux
/dev/nvme0n2p3      4196352 6291455 2095104 1023M  5 Extended
...
```
删除分区示例：
```
[root@redhat8 ~]# fdisk /dev/nvme0n2

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d
Partition number (1-3, default 3): 3
Partition 3 has been deleted.

Command (m for help): wq
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
创建一个扩展分区，并在其上面创建一个逻辑分区，并指定大小：
```
[root@redhat8 ~]# fdisk /dev/nvme0n2
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): e
Partition number (3,4, default 3): 
First sector (4196352-6291455, default 4196352): 
Last sector, +sectors or +size{K,M,G,T,P} (4196352-6291455, default 6291455): +1000M
Created a new partition 3 of type 'Extended' and of size 1000 MiB.

Command (m for help): n
Partition type
   p   primary (2 primary, 1 extended, 1 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 5
First sector (4198400-6244351, default 4198400): 
Last sector, +sectors or +size{K,M,G,T,P} (4198400-6244351, default 6244351): +512M
Created a new partition 5 of type 'Linux' and of size 512 MiB.

Command (m for help): wq
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
查看创建的分区：
```
[root@redhat8 ~]# fdisk -l
...
Device         Boot   Start     End Sectors  Size Id Type
/dev/nvme0n2p1         2048 2099199 2097152    1G 83 Linux
/dev/nvme0n2p2      2099200 4196351 2097152    1G 83 Linux
/dev/nvme0n2p3      4196352 6244351 2048000 1000M  5 Extended
/dev/nvme0n2p5      4198400 5246975 1048576  512M 83 Linux
...
```
修改分区类型：
```
[root@redhat8 ~]# fdisk /dev/nvme0n2

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): t
Partition number (1-3, default 3): 2
Hex code (type L to list all codes): L
...
Hex code (type L to list all codes): b
Changed type of partition 'Linux' to 'W95 FAT32'.

Command (m for help): wq   
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```
查看分区类型：
```
[root@redhat8 ~]# fdisk -l
...
Device         Boot   Start     End Sectors  Size Id Type
/dev/nvme0n2p1         2048 2099199 2097152    1G 83 Linux
/dev/nvme0n2p2      2099200 4196351 2097152    1G  b W95 FAT32
/dev/nvme0n2p3      4196352 6244351 2048000 1000M  5 Extended
/dev/nvme0n2p5      4198400 5246975 1048576  512M 83 Linux
```
如果在删除或修改后，分区表没更新，可以使用`partprobe`命令(RedHat8自动更新，此命令没试过)。

fdisk命令详细介绍及学习参考链接:[https://www.runoob.com/linux/linux-comm-fdisk.html](https://www.runoob.com/linux/linux-comm-fdisk.html)
### lsbik命令
命令示例如下,对比与`fdisk -l`的差异：：
```
[root@redhat8 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0            11:0    1  6.6G  0 rom  
nvme0n1       259:0    0   20G  0 disk 
├─nvme0n1p1   259:1    0    1G  0 part /boot
└─nvme0n1p2   259:2    0   19G  0 part 
  ├─rhel-root 253:0    0   17G  0 lvm  /
  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
nvme0n2       259:3    0    3G  0 disk 
├─nvme0n2p1   259:4    0    1G  0 part 
├─nvme0n2p2   259:5    0    1G  0 part 
├─nvme0n2p3   259:10   0    1K  0 part 
└─nvme0n2p5   259:11   0  512M  0 part 
```
### gdisk命令
gdisk用来划分GPT分区，可以划分容量大于2T的硬盘。操作和fdisk基本一致：
```
root@redhat8 ~]# gdisk /dev/nvme0n2
GPT fdisk (gdisk) version 1.0.3
Partition table scan:
  MBR: MBR only
  BSD: not present
  APM: not present
  GPT: not present
***************************************************************
Found invalid GPT and valid MBR; converting MBR to GPT format
in memory. THIS OPERATION IS POTENTIALLY DESTRUCTIVE! Exit by
typing 'q' if you don't want to convert your MBR partitions
to GPT format!
***************************************************************

Command (? for help): ?
```
## 文件系统管理
### mkfs命令
直接挂载上面创建的分区：
```
[root@redhat8 ~]# mount /dev/nvme0n2p5
mount: /dev/nvme0n2p5: can't find in /etc/fstab.
```
在新的逻辑分区上创建文件系统：
```
[root@redhat8 /]# chmod -R 755 /testfs
[root@redhat8 /]# mount /dev/nvme0n2p5 /testfs
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/nvme0n2p5         488M  416K  462M   1% /testfs
```
将`/dev/nvme0n2p5`格式化为ext3格式：
```
[root@redhat8 ~]# mkfs -t ext3 /dev/nvme0n2p5
mke2fs 1.44.3 (10-July-2018)
Creating filesystem with 131072 4k blocks and 32768 inodes
Filesystem UUID: 5c591d3b-5a89-4890-a57d-103e6ab2f77f
Superblock backups stored on blocks: 
	32768, 98304

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
将挂载定义添加到`/etc/fstab`文件中，尝试挂载（刚开始提示type不对，重启下再尝试就没问题了）：
```
[root@redhat8 /]# umount /testfs
[root@redhat8 /]# vi /etc/fstab
[root@redhat8 /]# tail -n 2 /etc/fstab
# /dev/sr0		/mnt			iso9660	noauto		0 0	
/dev/nvme0n2p5		/testfs			ext3	default		0 0
[root@redhat8 ~]# mount -a
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/nvme0n2p5         488M  416K  462M   1% /testfs
```
mkfs命令详细介绍:[https://www.runoob.com/linux/linux-comm-mkfs.html](https://www.runoob.com/linux/linux-comm-mkfs.html)

### blkid命令
使用UUID方式在`/etc/fstab`文件中挂载文件系统：
```
[root@redhat8 /]# blkid /dev/nvme0n2p5
/dev/nvme0n2p5: UUID="f3ad8520-7468-4ec3-938c-39a2fee46fef" SEC_TYPE="ext2" TYPE="ext3" PA
RTUUID="9711dc11-05"
[root@redhat8 ~]# tail -n 3 /etc/fstab
# /dev/sr0		/mnt			iso9660	noauto		0 0	
# /dev/nvme0n2p5		/testfs			ext3	defaults	0 0
UUID=f3ad8520-7468-4ec3-938c-39a2fee46fef	/testfs		ext3	defaults	0 
0
[root@redhat8 ~]# mount -a
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/nvme0n2p5         488M  416K  462M   1% /testfs
```
## 交换空间管理
查看并新建swap空间：
```
[root@redhat8 ~]# swapon
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G   0B   -2
[root@redhat8 ~]# swapon /dev/nvme0n2p6
[root@redhat8 ~]# swapon
NAME           TYPE      SIZE USED PRIO
/dev/dm-1      partition   2G   0B   -2
/dev/nvme0n2p6 partition 128M   0B   -3
[root@redhat8 ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1849432      682100      645680        9772      521652      988268
Swap:       2228216           0     2228216
```
关系新增的swap：
```
[root@redhat8 ~]# swapoff /dev/nvme0n2p6
[root@redhat8 ~]# swapon
NAME      TYPE      SIZE USED PRIO
/dev/dm-1 partition   2G   0B   -2
[root@redhat8 ~]# free
              total        used        free      shared  buff/cache   available
Mem:        1849432      682136      645624        9772      521672      988232
Swap:       2097148           0     2097148
```
swapon命令详细介绍:[https://www.runoob.com/linux/linux-comm-swapon.html](https://www.runoob.com/linux/linux-comm-swapon.html)         
mkswap命令详细介绍:[https://www.runoob.com/linux/linux-comm-mkswap.html](https://www.runoob.com/linux/linux-comm-mkswap.html)       
free命令详细介绍:[https://www.runoob.com/linux/linux-comm-free.html](https://www.runoob.com/linux/linux-comm-free.html)        
swapoff命令详细介绍:[https://www.runoob.com/linux/linux-comm-swapoff.html](https://www.runoob.com/linux/linux-comm-swapoff.html)
## 分区扩容
### 非LVM环境下分区
示例扩容`/dev/nvme0n2p5`:
```
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
/dev/nvme0n2p5         488M  416K  462M   1% /testfs
[root@redhat8 testfs]# ls
link1  ln1  lost+found  mount201031.log  mount.log  test1.log  test2.sh  test3.sh
[root@redhat8 testfs]# fdisk -l
...
Device         Boot   Start     End Sectors  Size Id Type
/dev/nvme0n2p1         2048 2099199 2097152    1G 83 Linux
/dev/nvme0n2p2      2099200 4196351 2097152    1G  b W95 FAT32
/dev/nvme0n2p3      4196352 6244351 2048000 1000M  5 Extended
/dev/nvme0n2p5      4198400 5246975 1048576  512M 83 Linux
[root@redhat8 /]# umount /testfs
[root@redhat8 /]# fdisk /dev/nvme0n2

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d
Partition number (1-3,5, default 5): 
Partition 5 has been deleted.

Command (m for help): n
Partition type
   p   primary (2 primary, 1 extended, 1 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 5
First sector (4198400-6244351, default 4198400): 
Last sector, +sectors or +size{K,M,G,T,P} (4198400-6244351, default 6244351): +700M
Created a new partition 5 of type 'Linux' and of size 700 MiB.
Partition #5 contains a ext3 signature.
Do you want to remove the signature? [Y]es/[N]o: N

Command (m for help): wq    
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
[root@redhat8 testfs]# fdisk -l
...
Device         Boot   Start     End Sectors  Size Id Type
/dev/nvme0n2p1         2048 2099199 2097152    1G 83 Linux
/dev/nvme0n2p2      2099200 4196351 2097152    1G  b W95 FAT32
/dev/nvme0n2p3      4196352 6244351 2048000 1000M  5 Extended
/dev/nvme0n2p5      4198400 5631999 1433600  700M 83 Linux
[root@redhat8 /]# mount /testfs
[root@redhat8 /]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/nvme0n2p5         488M  444K  462M   1% /testfs
[root@redhat8 ~]# ls /testfs
link1  ln1  lost+found  mount201031.log  mount.log  test1.log  test2.sh  test3.sh
```
&#8195;&#8195;虽然看到`/dev/nvme0n2p5`分区大小改变了，数据虽然没丢，但是挂载文件系统后大小还是原来的，重新启动系统后一样没改变（应该不需要重启），检查文件系统以及变更文件系统大小后，就可以了：
```
[root@redhat8 ~]# e2fsck -f /dev/nvme0n2p5
e2fsck 1.44.3 (10-July-2018)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/nvme0n2p5: 18/32768 files (0.0% non-contiguous), 6269/131072 blocks
[root@redhat8 ~]# resize2fs /dev/nvme0n2p5
resize2fs 1.44.3 (10-July-2018)
Resizing the filesystem on /dev/nvme0n2p5 to 179200 (4k) blocks.
The filesystem on /dev/nvme0n2p5 is now 179200 (4k) blocks long.
[root@redhat8 ~]# mount -a
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
...
tmpfs                  181M  4.0K  181M   1% /run/user/1000
/dev/nvme0n2p5         672M  568K  637M   1% /testfs
[root@redhat8 ~]# ls /testfs
link1  ln1  lost+found  mount201031.log  mount.log  test1.log  test2.sh  test3.sh
[root@redhat8 ~]# cat /testfs/ln1
link files test file
change the source file
```
### LVM环境下扩容
&#8195;&#8195;对于在VG中的逻辑卷扩容，如果VG容量足够，使用`lvextend`命令进行扩容，如果VG容量不够，划分新磁盘过来，创建新的分区，然后用`vgextend`命令先对VG进行扩容，然后再扩容逻辑卷。

参考文档:[https://blog.csdn.net/chongxin1/article/details/76072071/](https://blog.csdn.net/chongxin1/article/details/76072071/)
