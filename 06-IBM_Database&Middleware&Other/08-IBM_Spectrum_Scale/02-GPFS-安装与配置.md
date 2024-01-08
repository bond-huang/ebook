# GPFS-安装与配置
## GPFS安装
## AIX集群新建共享文件系统
### 磁盘分配
从存储将磁盘分配给两个AIX节点，两个节点上磁盘添加PVID：
```sh
chdev -l hdisk71 -a pv=yes
```
确保PVID一致，查看磁盘属性，不是no_reserve就修改：
```sh
chdev -l hdisk71 -a reserve_policy=no_reserve
```
### NSD配置
temp目录下创建文件gpfs_testapp，写入内容示例：
```
hdisk71:TEST-GPFS-server1:TEST-GPFS-server2:dataAndMetadata::testapp_nsd:
```
注意格式，写错了会报错，创建NSD命令示例：
```
# mmcrnsd -F /tmp/gpfs_testapp

mmcrnsd: Processing disk hdisk71
mmcrnsd: 6027-1371 Propagating the cluster configuration data to all
  affected nodes.  This is an asynchronous process.
```
运行后gpfs_testapp文件内容，后面还会用到：
```
# cat gpfs_testapp
# hdisk71:TEST-GPFS-server1:TEST-GPFS-server2:dataAndMetadata::testapp_nsd:
testapp_nsd:::dataAndMetadata:-1::system
```
lspv查看：
```
hdisk71         00fa3307b4a53e8f                    testapp_nsd
```
`mmlsnsd -m`查看：
```sh
 Disk name    NSD volume ID      Device         Node name                Remarks       
---------------------------------------------------------------------------------------
 testapp_nsd 0A009C1A658E8BDA   /dev/hdisk71   TEST-GPFS-server1        server node
 testapp_nsd 0A009C1A658E8BDA   /dev/hdisk71   TEST-GPFS-server2        server node
```
### 创建文件系统
创建命令示例：
```sh
mmcrfs /testapp /dev/testapp -F /tmp/gpfs_testapp -A yes -m 1 -M 2 -r 1 -R 2 --mount-priority 0 --inode-limit 2M
```
运行示例：
```
# mmcrfs /testapp /dev/testapp -F /tmp/gpfs_testapp -A yes -m 1 -M 2 -r 1 -R 2 --mount-priority 0 --inode-limit 2M

GPFS: 6027-531 The following disks of testapp will be formatted on node HEGUIGPFS1:
    testapp_nsd: size 512000 MB
GPFS: 6027-540 Formatting file system ...
GPFS: 6027-535 Disks up to size 4.5 TB can be added to storage pool system.
Creating Inode File
Creating Allocation Maps
Creating Log Files
Clearing Inode Allocation Map
Clearing Block Allocation Map
Formatting Allocation Map for storage pool system
GPFS: 6027-572 Completed creation of file system /dev/testapp.
mmcrfs: 6027-1371 Propagating the cluster configuration data to all
  affected nodes.  This is an asynchronous process.
```
两个节点都查看创建的文件系统配置：
```
# mmlsfs /dev/testapp
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
 --create-time      Fri Dec 29 17:55:46 2023 File system creation time
 -z                 no                       Is DMAPI enabled?
 -L                 4194304                  Logfile size
 -E                 yes                      Exact mtime mount option
 -S                 no                       Suppress atime mount option
 -K                 whenpossible             Strict replica allocation option
 --fastea           yes                      Fast external attributes enabled?
 --encryption       no                       Encryption enabled?
 --inode-limit      2097152                  Maximum number of inodes
 --log-replicas     0                        Number of log replicas
 --is4KAligned      yes                      is4KAligned?
 --rapid-repair     yes                      rapidRepair enabled?
 --write-cache-threshold 0                   HAWC Threshold (max 65536)
 -P                 system                   Disk storage pools in file system
 -d                 testapp_nsd          Disks in file system
 -A                 yes                      Automatic mount option
 -o                 none                     Additional mount options
 -T                 /testapp             Default mount point
 --mount-priority   0                        Mount priority
```
`cat /etc/filesystems`查看：
```
/testapp:
        dev             = /dev/testapp
        vfs             = mmfs
        nodename        = -
        mount           = mmfs
        type            = mmfs
        account         = false
        options         = rw,mtime,atime,dev=testapp
```
### 挂载文件系统
主节点执行挂载文件系统：
```sh
mmmount /testapp -a
```
运行示例：
```
# mmmount /testapp -a
Fri Dec 29 18:26:11 CST 2023: 6027-1623 mmmount: Mounting file systems ...
```
两个节点查看：
```
/dev/testapp    500.00    497.77    1%     4038     1% /testapp
```
编辑两个服务端节点的hosts，加入客户端信息：
```sh
10.10.11.240	testapp1
10.10.11.241	testapp2
```
查看fileset（和其他文件系统一样，就没新建，如果没有，就需要新建）：
```
# mmlsfileset /dev/testapp -L
Filesets in file system 'testapp':
Name                            Id      RootInode  ParentId Created                      InodeSpace      MaxInodes    AllocInodes Comment
root                             0              3        -- Fri Dec 29 17:55:48 2023        0              2097152         500032 root fileset
```
两个节点编辑`/etc/exports`，添加：
```ini
/testapp -rw,root=testapp1:testapp2
```
主备节点运行exportfs：
```sh
exportfs -a
```
### 客户端配置
客户端修改hosts，加入服务端信息：
```SH
#####GPFS######
10.10.15.128    GPFS_SVC
```
客户端查看服务端提供的NFS服务：
```
[root@testapp2 ~]# showmount -e HEGUIGPFS_SVC |grep tyjgckbx
Export list for HEGUIGPFS_SVC:
/testapp    (everyone)
```
编辑`/etc/fstab`，加入内容：
```
HEGUIGPFS_SVC:/testapp       /data              nfs     defaults        0 0
```
创建挂载点：
```sh
mkdir /data
chmod -R 755 /data
```
挂载文件系统
```sh
mount GPFS_SVC:/testapp /data
```
## 待补充