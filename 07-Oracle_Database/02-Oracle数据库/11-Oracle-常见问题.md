# Oracle-常见问题
## 数据库连接问题
### 报错ORA-00257
报错示例：
```
ORA-00257:archiver error.Connect internal only,until freed
```
Archive满了，首先检查文件系统，然后检查Archive存放位置的空间，通常是`DATA/`下：
```
[root@oracledb2 ~]# su - grid
[grid@oracledb2 ~]$ asmcmd
ASMCMD> lsdg
State    Type    Rebal  Sector  Block       AU  Total_MB  Free_MB  Req_mir_free_MB  Usable_file_MB  Offline_disks  Voting_files  Name
MOUNTED  HIGH    Y         512   4096  1048576      1024      628                0             209              2             Y  CRS/
MOUNTED  EXTERN  N         512   4096  1048576    307200      553                0             553              0             N  DATA/
```
如果`DATA/`满了就清理Archive。
## 待补充