# GPFS-常用操作
## AIX服务端文件系统扩容
### 需求说明
服务端查看文件系统：
```
# df -g
/dev/test_file   5072.00    655.45   88% 109956474    84% /test_file
```
已经超过80%，需要扩容。文件系统服务端是两个AIX系统节点。服务端查看文件系统NSD：
```
# mmlsdisk /dev/test_file -L
disk         driver   sector     failure holds    holds                                    storage
name         type       size       group metadata data  status        availability disk id pool         remarks   
------------ -------- ------ ----------- -------- ----- ------------- ------------ ------- ------------ ---------
test_file    nsd         512          -1 yes      yes   ready         up                 1 system        desc
test_file_nsd1 nsd         512          -1 yes      yes   ready         up                 2 system        desc
test_file_nsd2 nsd         512          -1 yes      yes   ready         up                 3 system        desc
test_file_nsd3 nsd         512          -1 yes      yes   ready         up                 4 system        desc
test_file_nsd4 nsd         512          -1 yes      yes   ready         up                 5 system        desc
Number of quorum disks: 5 
Read quorum value:      3
Write quorum value:     3
```
### 磁盘分配
从存储分配两块磁盘给两个AIX节点，两个节点上磁盘添加PVID：
```sh
chdev -l hdisk72 -a pv=yes
chdev -l hdisk73 -a pv=yes
```
确保PVID一致，查看磁盘属性，不是no_reserve就修改：
```sh
chdev -l hdisk72 -a reserve_policy=no_reserve
chdev -l hdisk73 -a reserve_policy=no_reserve
```
### NSD配置
temp目录下创建文件gpfs_test56，写入内容示例：
```
hdisk72:TEST-GPFS-server1:TEST-GPFS-server2:dataAndMetadata::test_file_nsd5:
hdisk73:TEST-GPFS-server1:TEST-GPFS-server2:dataAndMetadata::test_file_nsd6:
```
注意格式，写错了会报错，创建NSD命令示例：
```
# mmcrnsd -F gpfs_test56
mmcrnsd: Processing disk hdisk72
mmcrnsd: Processing disk hdisk73
mmcrnsd: 6027-1371 Propagating the cluster configuration data to all
  affected nodes.  This is an asynchronous process.
```
lspv查看：
```
hdisk72         00fa3307b59ff54d                    test_file_nsd5              
hdisk73         00fa3307b5a01fbb                    test_file_nsd6  
```
`mmlsnsd -m`查看：
```sh
 Disk name    NSD volume ID      Device         Node name                Remarks       
---------------------------------------------------------------------------------------
 test_file_nsd5 0A009C1A658EC1F5   /dev/hdisk72   TEST-GPFS-server1        server node
 test_file_nsd5 0A009C1A658EC1F5   /dev/hdisk72   TEST-GPFS-server2        server node
 test_file_nsd6 0A009C1A658EC1FE   /dev/hdisk73   TEST-GPFS-server1        server node
 test_file_nsd6 0A009C1A658EC1FE   /dev/hdisk73   TEST-GPFS-server2        server node
```
`mmlsnsd`命令查看：
```sh
 File system   Disk name    NSD servers                                    
---------------------------------------------------------------------------
 (free disk)   test_file_nsd5 TEST-GPFS-server1,TEST-GPFS-server2 
 (free disk)   test_file_nsd6 TEST-GPFS-server1,TEST-GPFS-server2
```
两个节点都可以看到。
### 将磁盘加入到文件系统
加入命令示例：
```sh
mmadddisk test_file -F /tmp/gpfs_test56  ##加-r是rebalance数据到新磁盘
```
运行示例：
```
# mmadddisk test_file -F /tmp/gpfs_test56

GPFS: 6027-531 The following disks of test_file will be formatted on node HEGUIGPFS2:
    test_file_nsd5: size 1048576 MB
    test_file_nsd6: size 1048576 MB
Extending Allocation Map
Checking Allocation Map for storage pool system
GPFS: 6027-1503 Completed adding disks to file system test_file.
mmadddisk: 6027-1371 Propagating the cluster configuration data to all
  affected nodes.  This is an asynchronous process.
```
查看文件系统磁盘：
```
# mmlsdisk /dev/test_file -L
disk         driver   sector     failure holds    holds                                    storage
name         type       size       group metadata data  status        availability disk id pool         remarks   
------------ -------- ------ ----------- -------- ----- ------------- ------------ ------- ------------ ---------
test_file    nsd         512          -1 yes      yes   ready         up                 1 system        desc
test_file_nsd1 nsd         512          -1 yes      yes   ready         up                 2 system        desc
test_file_nsd2 nsd         512          -1 yes      yes   ready         up                 3 system        desc
test_file_nsd3 nsd         512          -1 yes      yes   ready         up                 4 system        desc
test_file_nsd4 nsd         512          -1 yes      yes   ready         up                 5 system        desc
test_file_nsd5 nsd         512          -1 yes      yes   ready         up                 6 system        
test_file_nsd6 nsd         512          -1 yes      yes   ready         up                 7 system        
Number of quorum disks: 5 
Read quorum value:      3
Write quorum value:     3
```
服务端两个节点查看文件系统大小：
```
# df -g |grep test_file
/dev/test_file   7120.00   2703.28   63% 109957746    84% /test_file
```
### 客户端查看
服务端查看提供服务的客户端：
```
# cat /etc/exports |grep test
/test_file -rw,root=test-app1:test-app2:test-app3:test-app4
```
分别登录这些系统查看文件系统大小：
```
[root@test-app1 ~]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg_linux-LogVol00
                       98G   36G   58G  39% /
tmpfs                 7.9G   72K  7.9G   1% /dev/shm
/dev/vda1             477M   34M  419M   8% /boot
GPFS_SVC:/test_file
                      7.0T  4.4T  2.7T  63% /testfile
```
## Linux服务端文件系统扩容
### 服务端配置检查
服务端节点1磁盘配置：
```
[root@nfs1 ~]# lsblk
NAME                        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                          11:0    1 1024M  0 rom  
sda                           8:0    0  100G  0 disk 
├─sda1                        8:1    0  500M  0 part /boot
└─sda2                        8:2    0 99.5G  0 part 
  ├─vg_nfs1-lv_root (dm-0)  253:0    0   50G  0 lvm  /
  ├─vg_nfs1-lv_swap (dm-1)  253:1    0  7.8G  0 lvm  [SWAP]
  └─vg_nfs1-lv_home (dm-12) 253:12   0 41.7G  0 lvm  /home
sde                           8:64   0  500G  0 disk 
└─sde1                        8:65   0  500G  0 part 
sdf                           8:80   0  500G  0 disk 
└─sdf1                        8:81   0  500G  0 part 
sdb                           8:16   0  600G  0 disk 
└─sdb1                        8:17   0  600G  0 part 
sdc                           8:32   0  500G  0 disk 
└─sdc1                        8:33   0  500G  0 part 
sdd                           8:48   0  500G  0 disk 
└─sdd1                        8:49   0  500G  0 part 
```
服务端节点2磁盘配置：
```
[root@nfs2 ~]# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                         11:0    1 1024M  0 rom  
sda                          8:0    0  100G  0 disk 
├─sda1                       8:1    0  500M  0 part /boot
└─sda2                       8:2    0 99.5G  0 part 
  ├─vg_nfs2-lv_root (dm-0) 253:0    0   50G  0 lvm  /
  ├─vg_nfs2-lv_swap (dm-1) 253:1    0  7.8G  0 lvm  [SWAP]
  └─vg_nfs2-lv_home (dm-8) 253:8    0 41.7G  0 lvm  /home
sdb                          8:16   0  500G  0 disk 
└─sdb1                       8:17   0  500G  0 part 
sdc                          8:32   0  600G  0 disk 
└─sdc1                       8:33   0  600G  0 part 
sdd                          8:48   0  500G  0 disk 
└─sdd1                       8:49   0  500G  0 part 
sde                          8:64   0  500G  0 disk 
sdf                          8:80   0  500G  0 disk 
```
查看NSD配置：
```
[root@nfs1 ~]# mmlsnsd -m

 Disk name    NSD volume ID      Device         Node name                Remarks       
---------------------------------------------------------------------------------------
 nfs_nsd1     0A00033659DD7C41   /dev/dm-7      nfs1                     server node
 nfs_nsd1     0A00033659DD7C41   /dev/dm-2      nfs2                     server node
 nfs_nsd2     0A0003365B9F6423   /dev/dm-9      nfs1                     server node
 nfs_nsd2     0A0003365B9F6423   /dev/dm-5      nfs2                     server node
 nfs_nsd3     0A0003365BD9620B   /dev/dm-4      nfs1                     server node
 nfs_nsd3     0A0003365BD9620B   /dev/dm-3      nfs2                     server node
 nfs_nsd4     0A0003365FE449B4   /dev/sde       nfs1                     server node
 nfs_nsd4     0A0003365FE449B4   /dev/sde       nfs2                     server node
 nfs_nsd5     0A0003365FE449B7   /dev/sdf       nfs1                     server node
 nfs_nsd5     0A0003365FE449B7   /dev/sdf       nfs2                     server node
```
查看文件系统配置：
```
 [root@nfs1 ~]# mmlsnsd

 File system   Disk name    NSD servers                                    
---------------------------------------------------------------------------
 gpfs          nfs_nsd1     nfs1,nfs2                
 gpfs          nfs_nsd2     nfs1,nfs2                
 gpfs          nfs_nsd3     nfs1,nfs2                
 gpfs          nfs_nsd4     nfs1,nfs2                
 gpfs          nfs_nsd5     nfs1,nfs2  
```
配置简单，就一个文件系统，查看文件系统里面的NSD：
```
[root@nfs1 tmp]# mmlsdisk gpfs
disk         driver   sector     failure holds    holds                            storage
name         type       size       group metadata data  status        availability pool
------------ -------- ------ ----------- -------- ----- ------------- ------------ ------------
nfs_nsd1     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd2     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd3     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd4     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd5     nsd         512          -1 Yes      Yes   ready         up           system 
```
### 磁盘分配
&#8195;&#8195;将两块磁盘分配给两个服务节点，如果是VMware虚拟机，先分配给VMware再分配到两个服务节点，注意选择共享模式。节点1查看磁盘：
```
sdg                           8:96   0  600G  0 disk  
└─mpathh (dm-13)            253:13   0  600G  0 mpath 
sdh                           8:112  0  600G  0 disk  
└─mpathi (dm-14)            253:14   0  600G  0 mpath 
```
节点2查看磁盘：
```
节点2：
sdg                          8:96   0  600G  0 disk 
sdh                          8:112  0  600G  0 disk
```
节点1虽然是多路径，但是其实就一条，如果是多路径，可以使用dm-13这种格式磁盘。
### NSD配置
temp目录下创建文件gpfs_nfs_nsd6，写入内容示例：
```sh
/dev/sdg:nfs1:nfs2:dataAndMetadata::nfs_nsd6:
/dev/sdh:nfs1:nfs2:dataAndMetadata::nfs_nsd7:
```
创建NSD磁盘命令：
```sh
mmcrnsd -F /tmp/gpfs_nfs_nsd6
```
运行示例：
```
[root@nfs1 tmp]# mmcrnsd -F /tmp/gpfs_nfs_nsd6
mmcrnsd: Processing disk sdg
mmcrnsd: Processing disk sdh
mmcrnsd: Propagating the cluster configuration data to all
  affected nodes.  This is an asynchronous process.
```
运行后gpfs_nfs_nsd6文件内容：
```
[root@nfs1 tmp]# cat gpfs_nfs_nsd6
# /dev/sdg:nfs1:nfs2:dataAndMetadata::nfs_nsd6:
nfs_nsd6:::dataAndMetadata:-1::system
# /dev/sdh:nfs1:nfs2:dataAndMetadata::nfs_nsd7:
nfs_nsd7:::dataAndMetadata:-1::system
```
查看NSD：
```
[root@nfs1 tmp]# mmlsnsd -m

 Disk name    NSD volume ID      Device         Node name                Remarks       
---------------------------------------------------------------------------------------
 nfs_nsd1     0A00033659DD7C41   /dev/dm-7      nfs1                     server node
 nfs_nsd1     0A00033659DD7C41   /dev/dm-2      nfs2                     server node
 nfs_nsd2     0A0003365B9F6423   /dev/dm-9      nfs1                     server node
 nfs_nsd2     0A0003365B9F6423   /dev/dm-5      nfs2                     server node
 nfs_nsd3     0A0003365BD9620B   /dev/dm-4      nfs1                     server node
 nfs_nsd3     0A0003365BD9620B   /dev/dm-3      nfs2                     server node
 nfs_nsd4     0A0003365FE449B4   /dev/sde       nfs1                     server node
 nfs_nsd4     0A0003365FE449B4   /dev/sde       nfs2                     server node
 nfs_nsd5     0A0003365FE449B7   /dev/sdf       nfs1                     server node
 nfs_nsd5     0A0003365FE449B7   /dev/sdf       nfs2                     server node
 nfs_nsd6     0A000336658EB89C   /dev/sdg       nfs1                     server node
 nfs_nsd6     0A000336658EB89C   /dev/sdg       nfs2                     server node
 nfs_nsd7     0A000336658EB89F   /dev/sdh       nfs1                     server node
 nfs_nsd7     0A000336658EB89F   /dev/sdh       nfs2                     server node
```
查看文件系统关联信息(两个节点都可以看到)：
```
[root@nfs1 tmp]# mmlsnsd

 File system   Disk name    NSD servers                                    
---------------------------------------------------------------------------
 gpfs          nfs_nsd1     nfs1,nfs2                
 gpfs          nfs_nsd2     nfs1,nfs2                
 gpfs          nfs_nsd3     nfs1,nfs2                
 gpfs          nfs_nsd4     nfs1,nfs2                
 gpfs          nfs_nsd5     nfs1,nfs2                
 (free disk)   nfs_nsd6     nfs1,nfs2                
 (free disk)   nfs_nsd7     nfs1,nfs2   
```
### 磁盘加入到文件系统
将磁盘加入到文件系统中命令示例：
```sh
mmadddisk gpfs -F /tmp/gpfs_nfs_nsd6  # 加-r是rebalance数据到新磁盘
```
运行示例：
```
[root@nfs1 tmp]# mmadddisk gpfs -F /tmp/gpfs_nfs_nsd6

The following disks of gpfs will be formatted on node nfs2:
    nfs_nsd6: size 614400 MB
    nfs_nsd7: size 614400 MB
Extending Allocation Map
Checking Allocation Map for storage pool system
Completed adding disks to file system gpfs.
mmadddisk: Propagating the cluster configuration data to all
  affected nodes.  This is an asynchronous process.
```
查看是否加入：
```
[root@nfs1 tmp]# mmlsnsd

 File system   Disk name    NSD servers                                    
---------------------------------------------------------------------------
 gpfs          nfs_nsd1     nfs1,nfs2                
 gpfs          nfs_nsd2     nfs1,nfs2                
 gpfs          nfs_nsd3     nfs1,nfs2                
 gpfs          nfs_nsd4     nfs1,nfs2                
 gpfs          nfs_nsd5     nfs1,nfs2                
 gpfs          nfs_nsd6     nfs1,nfs2                
 gpfs          nfs_nsd7     nfs1,nfs2    
```
查看文件系统的磁盘：
```
[root@nfs1 tmp]# mmlsdisk gpfs
disk         driver   sector     failure holds    holds                            storage
name         type       size       group metadata data  status        availability pool
------------ -------- ------ ----------- -------- ----- ------------- ------------ ------------
nfs_nsd1     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd2     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd3     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd4     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd5     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd6     nsd         512          -1 Yes      Yes   ready         up           system       
nfs_nsd7     nsd         512          -1 Yes      Yes   ready         up           system 
```
客户端检查确认是否已经扩容：
```
[root@nfs1 tmp]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg_nfs1-lv_root
                       50G  6.5G   41G  14% /
tmpfs                 3.9G   80K  3.9G   1% /dev/shm
/dev/sda1             477M   34M  419M   8% /boot
/dev/mapper/vg_nfs1-lv_home
                       41G   48M   39G   1% /home
/dev/gpfs             3.8T  2.3T  1.5T  61% /gpfs
```
如果有客户端，也查看是否成功扩容。
## 文件系统配置修改
### --inode-limit配置修改
修改示例：
```
# mmchfs test_file --inode-limit 150M
Set maxInodes for inode space 0 to 157286400
Fileset root changed.
```
查看修改结果：
```
# mmlsfs /dev/test_file
flag                value                    description
------------------- ------------------------ -----------------------------------
 -f                 8192                     Minimum fragment size in bytes
 -i                 4096                     Inode size in bytes
 -I                 16384                    Indirect block size in bytes
 -m                 1                        Default number of metadata replicas
 -M                 2                        Maximum number of metadata replicas
 -r                 1                        Default number of data replicas
 -R                 2                        Maximum number of data replicas
 -j                 cluster                  Block allocation type
 -D                 nfs4                     File locking semantics in effect
 -k                 all                      ACL semantics in effect
 -n                 32                       Estimated number of nodes that will mount file system
 -B                 262144                   Block size
 -Q                 none                     Quotas accounting enabled
                    none                     Quotas enforced
                    none                     Default quotas enabled
 --perfileset-quota no                       Per-fileset quota enforcement
 --filesetdf        no                       Fileset df enabled?
 -V                 14.10 (4.1.0.4)          File system version
 --create-time      Tue Dec 12 17:01:34 2017 File system creation time
 -z                 no                       Is DMAPI enabled?
 -L                 4194304                  Logfile size
 -E                 yes                      Exact mtime mount option
 -S                 no                       Suppress atime mount option
 -K                 whenpossible             Strict replica allocation option
 --fastea           yes                      Fast external attributes enabled?
 --encryption       no                       Encryption enabled?
 --inode-limit      157286400                Maximum number of inodes
 --log-replicas     0                        Number of log replicas
 --is4KAligned      yes                      is4KAligned?
 --rapid-repair     yes                      rapidRepair enabled?
 --write-cache-threshold 0                   HAWC Threshold (max 65536)
 -P                 system                   Disk storage pools in file system
 -d                 test_file;test_file_nsd1;test_file_nsd2;test_file_nsd3;test_file_nsd4;test_file_nsd5;test_file_nsd6  Disks in file system
 -A                 yes                      Automatic mount option
 -o                 none                     Additional mount options
 -T                 /test_file               Default mount point
 --mount-priority   0                        Mount priority
```

## 待补充