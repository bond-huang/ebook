# RedHat-常见维护

## 磁盘维护
### 硬盘故障维护
#### 单盘对应单文件系统
&#8195;&#8195;在服务器上单盘对应单文件系统通常是分布式文件系统，大数据平台例如Hadoop基本是这样的，硬盘故障了文件系统就挂了了，但是不影响平台运行，更换即可，下面以更换Hadoop大数据平台故障硬盘为例。

查看挂载点，属精品一块盘对应一个挂载点，有规律可循，大致可以看出哪个掉了：
```sh
df -h
```
然后确认是哪个没挂上：
```sh
cat /etc/fstab
```
例如此次是`/mnt/disk13`，`/etc/fstab`里面屏蔽掉：
```sh
#UUID=ec2fd46a-0f79-4475-a34e-2776e604abc9 /mnt/disk13      ext4 defaults 0 0
```
物理更换硬盘，注意可能做了raid0，需要重启服务器重新做，启动后`lsblk`查看新磁盘：
```sh
sdm                                                      
└─sdm1 ext4         aefa5919-a395-4854-82ab-77c5fcdce7ee /mnt/disk12
sdn                                                      
sdo                                                      
└─sdo1 ext4         af417f60-d47a-476c-b4fe-9287b08e4139 /mnt/disk14
```
新盘是`sdn`，如果磁盘小于2T，使用fdisk创建分区：
```sh
fdisk /dev/sdn  # 依次n(创建)、p(primary)、回车(默认ID)、回车(默认大小)、w(保存)
# lsblk 查看生成了sdn1分区
mkfs.ext4 /dev/sdn1 # lsblk -f 查看有UUID生成
```
一般情况下磁盘都大于2T，需要用gdisk，示例：
```sh 
[root@node30 ~]# sudo parted /dev/sdn
GNU Parted 3.1
Using /dev/sdn
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt    # 交互式输入                                                  
Warning: The existing disk label on /dev/sdn will be destroyed and all data on this disk will be lost. Do you want to continue?                                                               
Yes/No? yes       # 交互式输入                                                         
(parted) mkpart primary ext4 1MiB 100%
(parted) print    # 交互式输入                                                       
Model: UN LOGICAL VOLUME (scsi)
Disk /dev/sdn: 2400GB 
Sector size (logical/physical): 512B/4096B
Partition Table: gpt  # 交互式输入
Disk Flags: 

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  2400GB  2400GB               primary

(parted) quit   # 交互式输入                                                        
Information: You may need to update /etc/fstab.
```
创建完成后lsblk能看到分区`sdn1`,创建文件系统：
```sh
mkfs.ext4 /dev/sdn1
```
`lsblk -f`可以看到UUID：
```sh
sdn                                                      
└─sdn1 ext4         487f1291-625b-4c0d-93a0-6fa0e51168a4
```
修改fstab，UUID换成新的，并且取消注释：
```sh
UUID=487f1291-625b-4c0d-93a0-6fa0e51168a4 /mnt/disk13     ext4 defaults 0 0 
```
挂载文件系统：
```sh
mount 487f1291-625b-4c0d-93a0-6fa0e51168a4 /mnt/disk13
```
确认挂载成功：
```sh
df -h
lsblk -f
```
注意事项：
- 如果是raid0需要停机更换硬盘重新做raid，注意一定要认真确认故障盘
- 如果系统需要重启，一定要在`/etc/fstab`屏蔽异常的文件系统，避免重启后系统挂载不了文件系统卡死
- 注意建议在`/etc/fstab`用UUID，挂载也是UUID，因为一对一的盘，下次更换盘符说不定变了，UUID比较准确

## 待补充