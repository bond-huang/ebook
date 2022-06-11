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
## 查找系统中的文件
### 搜索文件
&#8195;&#8195;系统管理员需要使用工具来搜索文件系统中符合特定条件的文件。可以在文件系统层次结构中搜索文件的两个常用命令：
- `locate`命令搜索预生成索引中的文件名或文件路径，并即时返回结果
- `find`命令通过爬取整个文件系统层次结构来实时搜索文件

### 根据名称查找文件
&#8195;&#8195;`locate`命令根据文件的名称或路径查找文件。这种方式速度比较快，是从`mlocate`数据库中查找这些信息。但是，该数据库不会实时更新（每日自动更新），意味着`locate`将找不到自上次数据库更新以来创建的文件。`root`用户可在任何时候通过发出`updatedb`命令来强制即时更新：
```
[root@redhat8 ~]# updatedb
```
&#8195;&#8195;`locate`命令限制非特权用户的结果。若要查看生成的文件名，用户必须对文件所在的目录具有搜索权限。示例可由`huang`读取的目录树中，搜索名称或路径中包含`passwd`的文件：
```
[huang@redhat8 ~]$ locate passwd
/etc/passwd
/etc/passwd-
/etc/pam.d/passwd
/etc/security/opasswd
...output omitted...
```
即使文件名或路径仅部分匹配搜索查询，也会返回结果：
```
[root@redhat8 ~]# locate image
/images
/etc/selinux/targeted/contexts/virtual_image_context
/home/huang/.local/share/libvirt/images
/images/debian_wheezy_amd64_standard.qcow2
...output omitted...
```
`-i`选项执行不区分大小写的搜索：
```
[huang@redhat8 ~]$ locate -i message |more
...output omitted...
/usr/share/vim/vim80/lang/zh_TW.UTF-8/LC_MESSAGES
/usr/share/vim/vim80/lang/zh_TW.UTF-8/LC_MESSAGES/vim.mo
/usr/share/vim/vim80/syntax/messages.vim
/var/log/messages
/var/log/messages-20220515
```
`-n`选项限制`locate`返回的搜索结果数量，返回的搜索结果限制为前五个匹配项：
```
[huang@redhat8 ~]$ locate -n 3 snow.png
/usr/share/icons/Adwaita/16x16/status/weather-snow.png
/usr/share/icons/Adwaita/22x22/status/weather-snow.png
/usr/share/icons/Adwaita/24x24/status/weather-snow.png
```
### 实时搜索文件
&#8195;&#8195;`find`命令通过在文件系统层次结构中执行实时搜索来查找文件。比`locate`慢，但准确度更高。此外，它还可以根据文件名以外的条件搜索文件，例如文件权限、文件类型、文件大小或修改时间：
- `find`命令使用执行搜索的用户帐户查看文件系统中的文件
- 调用`find`命令的用户必须具有要查看其内容的目录的读取和执行权限
- `find`命令的第一个参数是要搜索的目录。如果省略了目录参数，则`find`将从当前目录中开始搜索，并在任何子目录中查找匹配项
- 若要按文件名搜索文件，可使用`-name FILENAME`选项。使用此选项时，`find`将返回与`FILENAME`完全匹配的文件的路径
- 对于`find`命令，完整词语选项使用单个短划线，选项跟在路径名参数后

例如从`/`目录开始搜索名为`sshd_config`的文件：
```
[root@redhat8 ~]# find / -name sshd_config
/etc/ssh/sshd_config
/root/etcbackup/etc/ssh/sshd_config
```
&#8195;&#8195;可以使用通配符搜索文件名，并返回部分匹配的所有结果。使用通配符时，请务必将要查找的文件名用引号括起，以防止终端对通配符进行解译。示例从`/`目录开始搜索名称以`.sh`结尾的文件：
```
[root@redhat8 ~]# find / -name '*.sh'
/boot/grub2/i386-pc/modinfo.sh
/etc/X11/xinit/xinitrc.d/50-systemd-user.sh
/etc/X11/xinit/xinitrc.d/localuser.sh
/etc/profile.d/lang.sh
/etc/profile.d/colorgrep.sh
...output omitted...
```
示例在`/etc/`目录中搜索名称的任何位置上含有词语`pass`的文件：
```
[root@redhat8 ~]# find /etc -name '*pass*'
/etc/security/opasswd
/etc/pam.d/passwd
/etc/pam.d/gdm-password
/etc/pam.d/password-auth
/etc/passwd-
/etc/passwd
/etc/authselect/password-auth
```
&#8195;&#8195;要对所给文件名执行不区分大小写的搜索，可使用`-iname`选项，后面加上要搜索的文件名。示例如下：
```
[root@redhat8 ~]# find / -iname '*message*'
/proc/sys/net/core/message_burst
/proc/sys/net/core/message_cost
/python/git-2.29.2/po/build/locale/de/LC_MESSAGES
/python/git-2.29.2/po/build/locale/tr/LC_MESSAGES
...output omitted...
```
#### 根据所有权或权限搜索文件
&#8195;&#8195;`find`可以根据所有权或权限搜索文件。按所有者搜索时的有用选项为`-user`和`-group`（按名称搜索），以及 `-uid`和`-giz`（按ID搜索）。示例在`/home/huang`目录中搜索由`root`拥有的文件：
```
[huang@redhat8 ~]$ find -user root
./Downloads/usr
./Downloads/usr/include
./Downloads/usr/include/c++
./Downloads/usr/include/c++/8
...output omitted...
```
示例在`/home/huang`目录中搜索由`root`组拥有的文件：
```
[huang@redhat8 ~]$ find -group root |more
./Downloads/usr
./Downloads/usr/include
./Downloads/usr/include/c++
./Downloads/usr/include/c++/8
...output omitted...
```
&#8195;&#8195;`-user`和`-group`选项可以一起使用，以搜索文件所有者和组所有者不同的文件。示例列出由用户`root`所有并且附属于`mail`组的文件：
```
[root@redhat8 ~]# find / -user root -group mail
/var/spool/mail
```
&#8195;&#8195;`-perm`选项用于查找具有特定权限集的文件。权限可以描述为八进制值，包含代表读取、写入和执行的`4`、`2`和`1`的某些组合。权限前面可以加上`/`或`-`符号：
- 前面带有`/`的数字权限将匹配文件的用户、组、其他人权限集中的至少一位
- 权限为`r--r--r--`的文件并不匹配`/222`，权限为`rw-r--r--`的文件才匹配
- 权限前带有`-`符号表示该位的所有三个实例都必须存在，因此前面的两个示例都不匹配，但诸如`rw-rw-rw-` 的对象则匹配

&#8195;&#8195;示例命令将匹配用户具有读取、写入和执行权限，组成员具有读取和写入权限且其他人具有只读权限的任何文件：
```
[root@redhat8 ~]# find /home -perm 764
```
&#8195;&#8195;示例匹配用户至少具有写入和执行权限，并且组至少具有写入权限，并且其他人至少具有读取权限的文件：
```
[root@redhat8 ~]# find /home -perm -324
/home/huang/.config/libvirt/storage/autostart/default.xml
```
&#8195;&#8195;示例匹配用户具有读取权限，或者组至少具有读取权限，或者其他人至少具有写入权限的文件：
```
[root@redhat8 ~]# find /home -perm /442
/home
/home/huang
/home/huang/.mozilla
...output omitted...
```
&#8195;&#8195;与`/`或`-`一起使用时，`0`值类似于通配符，因为其表示至少无任何内容的权限。示例匹配`/home/huang`目录中其他人至少具有读取权限的任何文件：
```
[huang@redhat8 ~]$ find -perm -004 |more
.
./.mozilla
./.mozilla/extensions
...output omitted...
```
示例在`/home/huang`目录中查找其他人拥有写入权限的所有文件：
```
[huang@redhat8 ~]$ find -perm -002 |more
.
./.mozilla
./.mozilla/extensions
./.mozilla/plugins
./.bash_logout
...output omitted...
```
#### 根据大小搜索文件
&#8195;&#8195;`find`命令可以查找与指定的大小相符的文件，该大小是通过`-size`选项加上数字值与单位来指定的。作`-size`选项的单位：
- `k`，表示千字节
- `M`，表示兆字节
- `G`，表示千兆字节 

示例显示如何搜索大小为`3`兆字节（向上取整）的文件：
```
[huang@redhat8 ~]$ find -size 3M
./Downloads/libstdc++-devel-8.2.1-3.5.el8.x86_64.rpm
```
搜索大小超过`1`千兆字节的文件：
```
[root@redhat8 ~]# find / -size +1G
/proc/kcore
```
列出大小不到`10KB`的所有文件：
```
[huang@redhat8 ~]$ find -size -10k
.
./.mozilla
./.mozilla/extensions
./.mozilla/plugins
...output omitted...
```
&#8195;&#8195;`-size`选项单位修饰符将所有内容向上取整为一个单位。例如，`find -size 1M`命令将显示小于`1MB`的文件，因为它将所有文件都向上取整为`1MB`。
#### 根据修改时间搜索文件
&#8195;&#8195;`-mmin`选项加上以分钟表示的时间，将搜索内容在过去`n`分钟前更改的所有文件。文件的时间戳始终向下舍入。与范围（`+n`和`-`）一起使用时支持分数值。示例查找文件内容在`180`分钟以前更改的所有文件：
```
[huang@redhat8 ~]$ find ./test -mmin 180
```
分钟数前加上`+`修饰符将查找在`n`分钟以前修改过的所有文件。示例：
```
[huang@redhat8 ~]$ find ./test -mmin +10
./test/cltopinfo
./test/test1
./test/clshowsrv
./test/clshowsrv1
...output omitted...
```
&#8195;&#8195;`-`修饰符则将搜索改为查找目录中在过去`n`分钟内更改的所有文件。示例过去十分钟内修改文件：
```
[huang@redhat8 ~]$ find ./test -mmin -10
./test
./test/testfind
```
#### 根据文件类型搜索文件
&#8195;&#8195;`find`命令中的`-type`选项将搜索范围限制为给定的文件类型。使用以下列表传递所需的标志以限制搜索范围：
- `f`，表示普通文件
- `d`，表示目录
- `l`，表示软链接
- `b`，表示块设备 

示例搜索`/home/huang`目录中的所有目录：
```
[huang@redhat8 ~]$ find -type d
.
./.mozilla
./.mozilla/extensions
./.mozilla/plugins
...output omitted...
```
搜索所有软连接：
```
[huang@redhat8 ~]$ find -type l
./.config/libvirt/storage/autostart/default.xml
```
`/dev`目录中所有块设备的列表：
```
[root@redhat8 ~]# find /dev -type b
/dev/dm-1
/dev/dm-0
/dev/sr0
/dev/nvme0n2p6
/dev/nvme0n2p5
...output omitted...
```
`-links`选项加上数字将查找具有特定硬链接数的所有文件：
- 数字前面带有`+`修饰符将查找硬链接数超过所给数目的文件
- 如果数字前面是`-`修饰符，则搜索将限制为硬链接数小于所给数目的所有文件

示例搜索硬链接数大于`3`的所有普通文件：
```
[root@redhat8 ~]# find / -type f -links +3 |more
/usr/bin/sha1hmac
/usr/bin/sha224hmac
/usr/bin/sha256hmac
/usr/bin/sha384hmac
/usr/bin/sha512hmac
/usr/sbin/e2fsck
/usr/sbin/fsck.ext2
...output omitted...
```
## 练习