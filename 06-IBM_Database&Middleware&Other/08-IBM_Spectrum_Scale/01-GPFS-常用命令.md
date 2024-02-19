# GPFS-常用命令
官方命令参考文档：[GPFS 5.1.8 Command reference](https://www.ibm.com/docs/en/storage-scale/5.1.8?topic=command-reference)
## 常用命令链接
常用命令官方参考链接：
- [mmcrfileset command](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=commands-mmcrfileset-command)
- [mmmount command](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=commands-mmmount-command)
- [mmdeldisk command](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=commands-mmdeldisk-command)
- [mmchfs command](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=commands-mmchfs-command#mmchfs)
- [mmcrfs command](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=commands-mmcrfs-command)
- [mmadddisk command](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=commands-mmadddisk-command)
- [mmlsfs command](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=commands-mmlsfs-command)

## 查看命令
### 集群查看
mmlscluster命令查看集群配置状态示例：
```
[root@nfs1 ~]# mmlscluster

GPFS cluster information
========================
  GPFS cluster name:         test.nfs1
  GPFS cluster id:           1220109251383892414
  GPFS UID domain:           test.nfs1
  Remote shell command:      /usr/bin/ssh
  Remote file copy command:  /usr/bin/scp
  Repository type:           CCR

 Node  Daemon node name  IP address  Admin node name  Designation
------------------------------------------------------------------
   1   nfs1              10.10.13.54   nfs1             quorum-manager
   2   nfs2              10.10.13.55   nfs2             quorum-manager
```
查看GPFS配置信息：
```
clusterName test.nfs1
clusterId 1220109251483892714
autoload yes
dmapiFileHandleSize 32
minReleaseLevel 4.1.0.4
ccrEnabled yes
tiebreakerDisks nfs_nsd1
adminMode central

File systems in cluster test.nfs1:
----------------------------------------
/dev/gpfs
```
### NSD查看
NSD查看命令示例：
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
NSD与文件系统关联信息查看示例：
```
[root@nfs1 ~]# mmlsnsd -L

 File system   Disk name    NSD volume ID      NSD servers                                   
---------------------------------------------------------------------------------------------
 gpfs          nfs_nsd1     0A00033659DD7C41   nfs1,nfs2                
 gpfs          nfs_nsd2     0A0003365B9F6423   nfs1,nfs2                
 gpfs          nfs_nsd3     0A0003365BD9620B   nfs1,nfs2                
 gpfs          nfs_nsd4     0A0003365FE449B4   nfs1,nfs2                
 gpfs          nfs_nsd5     0A0003365FE449B7   nfs1,nfs2  
```
### 文件系统查看
文件系统关联磁盘信息查看：
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
## 配置命令
NSD创建命令：[Create GPFS Network Shared Disks (NSD)](https://www.ibm.com/docs/en/storage-scale/4.2.0?topic=clusters-create-gpfs-network-shared-disks-nsd)
## 待补充