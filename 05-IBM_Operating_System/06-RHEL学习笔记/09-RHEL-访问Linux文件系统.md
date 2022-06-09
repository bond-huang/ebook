# RHEL-访问Linux文件系统
## 识别文件系统和设备
### 存储管理概念
&#8195;&#8195;Linux服务器上的文件是按文件系统层次结构（一个颠倒的目录树）访问的。该文件系统层次结构则是由系统可用的存储设备所提供的文件系统组装而来。每个文件系统都是一个已格式化的存储设备，可用于存储文件。
#### 文件系统和挂载点
&#8195;&#8195;要让文件系统的内容在文件系统层次结构中可用，必须将它挂载到一个空目录上。该目录被称为挂载点。挂载后，如果使用`ls`列出该目录，就会看到已挂载文件系统的内容，并可以正常访问和使用这些文件。许多文件系统都会作为启动进程的一部分自动挂载。
#### 文件系统、存储和块设备
&#8195;&#8195;在Linux中，对存储设备的低级别访问是由一种称为块设备的特殊类型文件提供的。在挂载这些块设备前，必须先使用文件系统对其进行格式化：
- 块设备文件与其他的设备文件一起存储在`/dev`目录中
- 设备文件是由操作系统自动创建的
- 在RHEL中，检测到的第一个`SATA/PATA`、`SAS`、`SCSI`或`USB`硬盘驱动器被称为`/dev/sda`，第二个被称为`/dev/sdb`，以此类推。这些名称代表整个硬盘驱动器
- 其他类型的存储设备有另外的命名方式
- 许多虚拟机都采用了较新的`virtio-scsi`超虚拟化存储，对应的命名形式为`/dev/sd*`

块设备命名如下表

设备类型|设备命名模式
:---|:---
SATA/SAS/USB附加存储|/dev/sda、/dev/sdb ...
virtio-blk超虚拟化存储（部分虚拟机）|/dev/vda、/dev/vdb ...
NVMe附加存储（很多 SSD）|/dev/nvme0, /dev/nvme1 ...
SD/MMC/eMMC存储（SD卡）|/dev/mmcblk0, /dev/mmcblk1 ... 

#### 磁盘分区
&#8195;&#8195;通常不会将整个存储设备设为一个文件系统。存储设备通常划分为更小的区块，称为分区。分区用于划分硬盘：
- 不同的部分可以通过不同的文件系统进行格式化或用于不同的用途
- 例如，一个分区可以包含用户主目录，另一个分区则可包含系统数据和日志
- 如果用户在主目录分区中填满了数据，系统分区可能依然有可用的空间

分区本身就是块设备：
- 在`SATA`附加存储中，
    - 第一磁盘上的第一个分区是`/dev/sda1`
    - 第二磁盘上的第三个分区是`/dev/sdb3`，以此类推
    - 超虚拟化存储设备采用了类似的命名体系
- `NVMe`附加 SSD 设备命名分区的方式却有所不同：
    - 其第一磁盘上的第一个分区是`/dev/nvme0p1`
    - 第二磁盘上的第三个分区是`/dev/nvme1p3`，以此类推
    - `SD`或`MMC`卡采用了类似的命名体系

host上`/dev/nvme0n2p1`设备文件的长列表显示其特殊文件类型为`b`，代表块设备：
```
[root@redhat8 ~]# ls -l /dev/nvme0n2p1
brw-rw----. 1 root disk 259, 6 Jun  6 21:43 /dev/nvme0n2p1
```
#### 逻辑卷
&#8195;&#8195;整理磁盘和分区的另一种方式是通过逻辑卷管理(LVM)。通过`LVM`，一个或多个块设备可以汇集为一个存储池，称为卷组。然后，卷组中的磁盘空间被分配到一个或多个逻辑卷，它们的功能等同于驻留在物理磁盘上的分区。`LVM`系统在创建时为卷组和逻辑卷分配名称：
- `LVM`在`/dev`中创建一个名称与组名匹配的目录，然后在该新目录中创建一个与逻辑卷同名的符号链接。之后，可以挂载该逻辑卷文件
- 例如，如果一个卷组名为`myvg`，其中有一个名为`mylv`的逻辑卷，那么其逻辑卷设备文件的完整路径名为`/dev/myvg/mylv`

### 检查文件系统
&#8195;&#8195;若要对本地和远程文件系统设备及可用空间大小有个简略了解，可以运行`df`命令。不带参数运行`df`时，它会报告所有已挂载的普通文件系统的总磁盘空间、已用磁盘空间、可用磁盘空间，以及已用磁盘空间占总磁盘空间的百分比。它会同时报告本地和远程文件系统。示例：
```
[root@redhat8 ~]# df
Filesystem            1K-blocks     Used Available Use% Mounted on
devtmpfs                 909368        0    909368   0% /dev
tmpfs                    924716        0    924716   0% /dev/shm
tmpfs                    924716     1372    923344   1% /run
tmpfs                    924716        0    924716   0% /sys/fs/cgroup
/dev/mapper/rhel-root  37734400 13701212  24033188  37% /
/dev/nvme0n1p1          1038336   172968    865368  17% /boot
/dev/nvme0n2p5           688048      568    651644   1% /testfs
tmpfs                    184940       16    184924   1% /run/user/42
tmpfs                    184940        4    184936   1% /run/user/0
```
文件系统说明：
- 示例host系统上的分区显示了两个物理文件系统，它们挂载于`/`和`/boot`。这对于虚拟机而言很常见
- `tmpfs`和`devtmpfs`设备是系统内存中的文件系统。在系统重启后，写入`tmpfs`或`devtmpfs`的文件都会消失

&#8195;&#8195;若要改善输出大小的可读性，可以使用两个不同的用户可读选项，即`-h`或`-H`。这两个选项的区别是：
- 使用`-h`时报告单位是`KiB`、`MiB`或`GiB`，分别是2的10、20及30次方
- 使用`-H`选项时报告单位是`SI`单位，即`KB`、`MB`或`GB`分别是10的3、6及9次方
- 硬盘驱动器制造商在广告其产品时通常使用`SI`单位

示例如下：
```
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               889M     0  889M   0% /dev
tmpfs                  904M     0  904M   0% /dev/shm
tmpfs                  904M  1.4M  902M   1% /run
tmpfs                  904M     0  904M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   36G   14G   23G  37% /
/dev/nvme0n1p1        1014M  169M  846M  17% /boot
/dev/nvme0n2p5         672M  568K  637M   1% /testfs
tmpfs                  181M   16K  181M   1% /run/user/42
tmpfs                  181M  4.0K  181M   1% /run/user/0
```
&#8195;&#8195;如需有关某一特定目录树使用的空间的详细信息，可以使用`du`命令。du 命令具有`-h`和`-H`选项，可以将输出转换为可读的格式。`du`命令以递归方式显示当前目录树中所有文件的大小。示例：
```
[root@redhat8 ~]# du /home/huang
...output omitted...
9080	/home/huang/.pyenv
0	/home/huang/umasktest
0	/home/huang/rsyctest
57660	/home/huang
```
以可读的格式显示上面示例目录的磁盘使用报告：
```
[root@redhat8 ~]# du -h /home/huang
...output omitted...
8.9M	/home/huang/.pyenv
0	/home/huang/umasktest
0	/home/huang/rsyctest
57M	/home/huang
```
## 挂载和卸载文件系统
### 手动挂载文件系统
&#8195;&#8195;驻留于可移动存储设备上的文件系统需要挂载后才能访问。`mount`命令允许`root`用户手动挂载文件系统。命令说明：
- `mount`命令的第一个参数指定要挂载的文件系统
- 第二个参数指定在文件系统层次结构中用作挂载点的目录

有两种常用方法可以为`mount`命令指定磁盘分区的文件系统：
- 在`/dev`的设备文件名称中包含文件系统
- 将`UUID`（一个通用唯一标识符）写入文件系统

挂载设备相对较为简单。需要识别要挂载的设备，确保挂载点存在，然后将设备挂载到挂载点上。
#### 识别块设备
&#8195;&#8195;每次连接到系统时，热插拔存储设备（不管是服务器caddy中的硬盘驱动器(HDD)或固态设备 (SSD)，还是USB存储设备）都可能插接到不同的端口上。使用`lsblk`命令可列出指定块设备或所有可用设备的详细信息：
```
[root@redhat8 ~]# lsblk
NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0            11:0    1 1024M  0 rom  
nvme0n1       259:0    0   40G  0 disk 
├─nvme0n1p1   259:1    0    1G  0 part /boot
├─nvme0n1p2   259:2    0   19G  0 part 
│ ├─rhel-root 253:0    0   36G  0 lvm  /
│ └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]
├─nvme0n1p3   259:3    0    1K  0 part 
└─nvme0n1p5   259:4    0   20G  0 part 
  └─rhel-root 253:0    0   36G  0 lvm  /
nvme0n2       259:5    0    3G  0 disk 
├─nvme0n2p1   259:6    0    1G  0 part 
├─nvme0n2p2   259:7    0    1G  0 part 
├─nvme0n2p3   259:8    0    1K  0 part 
├─nvme0n2p5   259:9    0  700M  0 part /testfs
└─nvme0n2p6   259:10   0  100M  0 part 
```
&#8195;&#8195;如果知道自己刚添加了一台有多个分区的`3GB`存储设备从上面的输出中`/dev/nvme0n2p1`等就是要挂载的分区。
#### 按块设备名称挂载
示例在`/mnt/testfs`目录上`/dev/nvme0n2p5`分区中挂载文件系统：
```
[root@redhat8 ~]# mount /dev/nvme0n2p5 /mnt/testfs
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
...output omitted...
/dev/nvme0n2p5         672M  568K  637M   1% /mnt/testfs
[root@redhat8 ~]# ls -l /testfs
total 4
-rw-r--r--. 1 root root 10 Jun  7 22:02 test.sh
[root@redhat8 ~]# ls -l /mnt/testfs
total 44
...output omitted...
-rw-r--r--. 1 root root   276 Oct 31  2020 test3.sh
```
示例说明：
- 若要挂载文件系统，目标目录必须已存在。默认情况下，`/mnt`目录存在并用作临时挂载点
- 可以将`/mnt`目录用作临时挂载点（创建`/mnt`的一个子目录会更好），除非有充分的理由将它挂载到文件系统层次结构中的特定位置

注意事项：
- 如果用作挂载点的目录不为空，则在挂载文件系统前复制到此目录中的任何文件均不可访问，直到将该文件系统再次卸载
- 这种方法在短期内可以正常工作。但是，如果从系统中添加或删除了设备，则操作系统检测磁盘的顺序可能会发生变化
- 这将更改与该存储设备关联的设备名称。更好的方法是通过内置于文件系统的某些特性进行挂载

示例在`/testfs`目录中写入文件，然后挂载文件系统（有其他数据），再次卸载：
```
[root@redhat8 ~]# ls -l /testfs
total 4
-rw-r--r--. 1 root root 10 Jun  7 22:02 test.sh
[root@redhat8 ~]# mount /dev/nvme0n2p5 /testfs
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
...output omitted...
/dev/nvme0n2p5         672M  568K  637M   1% /testfs
[root@redhat8 ~]# ls -l /testfs
total 44
...output omitted...
-rw-r--r--. 1 root root    50 Oct 31  2020 test1.log
-rw-r--r--. 1 root root   183 Oct 31  2020 test2.sh
-rw-r--r--. 1 root root   276 Oct 31  2020 test3.sh
[root@redhat8 ~]# umount /dev/nvme0n2p5 /testfs
umount: /testfs: not mounted.
[root@redhat8 ~]# ls -l /testfs
total 4
-rw-r--r--. 1 root root 10 Jun  7 22:02 test.sh
```
#### 按文件系统UUID挂载
&#8195;&#8195;一个稳定且与文件系统关联的标识符是`UUID`，这是一个非常长的十六进制数字，用作通用唯一标识符。该`UUID`是文件系统的一部分，只要文件系统没有重新创建过，它就会保持不变。`lsblk -fp`命令会列出设备的完整路径、其 UUID 和挂载点，以及分区中文件系统的类型。如果未挂载文件系统，挂载点将为空。示例：
```
[root@redhat8 ~]# lsblk -fp
NAME                  FSTYPE LABEL UUID                                   MOUNTPOINT
/dev/sr0                                                                  
/dev/nvme0n1                                                              
├─/dev/nvme0n1p1      xfs          0a510af6-8abf-4e64-b559-902804c93568   /boot
├─/dev/nvme0n1p2      LVM2_m       JIsKJl-M87D-CZHf-1SUD-Jo7p-jaKG-ZqWR0x 
│ ├─/dev/mapper/rhel-root
│ │                   xfs          de085a1b-1289-450c-b62b-c95f7a0918d3   /
│ └─/dev/mapper/rhel-swap
│                     swap         8384993e-b65f-4953-b727-c6db639e6c12   [SWAP]
├─/dev/nvme0n1p3                                                          
└─/dev/nvme0n1p5      LVM2_m       0RcVoT-1YY2-HOZ2-jXAd-BKdK-OJTP-SUOFaf 
  └─/dev/mapper/rhel-root
                      xfs          de085a1b-1289-450c-b62b-c95f7a0918d3   /
/dev/nvme0n2                                                              
├─/dev/nvme0n2p1                                                          
├─/dev/nvme0n2p2                                                          
├─/dev/nvme0n2p3                                                          
├─/dev/nvme0n2p5      ext3         f3ad8520-7468-4ec3-938c-39a2fee46fef   
└─/dev/nvme0n2p6  
```
根据文件系统的`UUID`挂载文件系统示例：
```
[root@redhat8 ~]# mount UUID="f3ad8520-7468-4ec3-938c-39a2fee46fef" /testfs
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
...output omitted...
/dev/nvme0n2p5         672M  568K  637M   1% /testfs
[root@redhat8 ~]# ls -l /test/fs
ls: cannot access '/test/fs': No such file or directory
[root@redhat8 ~]# ls -l /testfs
total 44
...output omitted...
-rw-r--r--. 1 root root   276 Oct 31  2020 test3.sh
```
### 自动挂载可移动存储设备
&#8195;&#8195;如果已登录并且使用的是图形桌面环境，则在插入任何可移动存储介质时，它将自动挂载。 
可移动存储设备将挂载到`/run/media/USERNAME/LABEL`，其中`USERNAME`是登录图形环境的用户名，而 `LABEL`是一个标识符，通常是创建时给文件系统取的名称（如果存在）。在移除设备之前，应手动将它卸载。
### 卸载文件系统
&#8195;&#8195;关机和重新引导过程会自动卸载所有文件系统。作为此过程的一部分，缓存在内存中的任何文件系统数据都会刷新到存储设备，从而确保文件系统不会遭受数据损坏。注意事项：
- 文件系统数据通常缓存在内存中。因此，为了避免损坏磁盘上的数据，务必先卸载可移动驱动器，然后再拔下它们
- 卸载过程会在释放驱动器之前同步数据，以确保数据完整性

`umount`命令卸载文件系统，需要使用挂载点作为参数：
```
[root@redhat8 ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
...output omitted...
/dev/nvme0n2p5         672M  568K  637M   1% /testfs
[root@redhat8 ~]# umount /testfs
```
分区写上也可以：
```
[root@redhat8 ~]# umount /dev/nvme0n2p5 /testfs
umount: /testfs: not mounted.
```
&#8195;&#8195;如果挂载的文件系统正在使用之中，则无法卸载。要成功执行`umount`命令，所有进程都需要停止访问挂载点下的数据。示例文件系统正在使用中（shell正将`/testfs`用作其当前工作目录），`umount`将失败，并生成错误消息：
```
[root@redhat8 ~]# cd /testfs
[root@redhat8 testfs]# pwd
/testfs
[root@redhat8 testfs]# umount /testfs
umount: /testfs: target is busy.
```
&#8195;&#8195;`lsof`命令列出所给目录中所有打开的文件以及访问它们的进程。识别哪些进程正在阻止文件系统被成功卸载非常有用。示例：
```
[root@redhat8 testfs]# lsof /testfs
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    7487 root  cwd    DIR  259,9     4096    2 /testfs
lsof    8021 root  cwd    DIR  259,9     4096    2 /testfs
lsof    8022 root  cwd    DIR  259,9     4096    2 /testfs
```
## 待补充