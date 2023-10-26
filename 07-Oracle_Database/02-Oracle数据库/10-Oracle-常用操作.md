# Oracle-常用操作
## Archive操作
### 清理Archive
首先查看Archive的位置，示例：
```
[root@oracledb2 ~]# su - oracle
[oracle@oracledb2 ~]$ sqlplus / as sysdba
......
SQL> archive log list
Database log mode              Archive Mode
Automatic archival             Enabled
Archive destination            +DATA/arch
Oldest online log sequence     14150
Next log sequence to archive   14150
Current log sequence           14153
SQL> exit
```
在`+DATA/arch`位置，查看`DATA`空间及查看目录下archive文件：
```
[root@oracledb2 ~]# su - grid
[grid@oracledb2 ~]$ asmcmd
ASMCMD> lsdg
State    Type    Rebal  Sector  Block       AU  Total_MB  Free_MB  Req_mir_free_MB  Usable_file_MB  Offline_disks  Voting_files  Name
MOUNTED  HIGH    Y         512   4096  1048576      1024      628                0             209              2             Y  CRS/
MOUNTED  EXTERN  N         512   4096  1048576    307200      553                0             553              0             N  DATA/
ASMCMD> cd DATA
ASMCMD> cd arch
ASMCMD> ls -l 
```
清理前喝多查看：
```
[root@oracledb2 ~]# su - oracle
[oracle@oracledb2 ~]$ rman target /
RMAN> crosscheck archivelog all;
RMAN> list expired archivelog all;
RMAN> list archivelog all;
```
保留七天的数据，进行清理
```
RMAN> delete noprompt archivelog until time 'sysdate-7';
```
如果清理后，空间没释放多少，还是很大，检查目录：
```
[root@oracledb2 ~]# su - grid
[grid@oracledb2 ~]$ asmcmd
ASMCMD> cd DATA
ASMCMD> ls
OAORCL/
arch/
ASMCMD> du arch
ASMCMD> du OAORCL
Used_MB      Mirror_used_MB
 306525              306525
ASMCMD> du    
Used_MB      Mirror_used_MB
 306525              306525
ASMCMD> cd OAORCL
ASMCMD> du ARCHIVELOG
Used_MB      Mirror_used_MB
 166632              166632
ASMCMD> cd arch
ASMCMD> ls -l 
```
&#8195;&#8195;`du arch`没反应，`du OAORCL`大小和`du DATA`差不多，并且`OAORCL`下的`ARCHIVELOG`很大，说明查看里面内容，历史归档日志很多：
```
[root@oracledb2 /]# su - grid
[grid@oracledb2 ~]$ asmcmd
ASMCMD> cd DATA
ASMCMD> cd OAORCL
ASMCMD> cd ARCHIVELOG
ASMCMD> ls
2021_01_01/
2021_01_02/
......
```
&#8195;&#8195;并且`ARCHIVELOG`目录里面包含日期内容与`arch`里面有重复，检查`ls -l arch`里面全是如下链接说明`arch`里面内容链接到了` +DATA/OAORCL/ARCHIVELOG`：
```
N    2_14171_949173618.dbf => +DATA/OAORCL/ARCHIVELOG/2023_10_26/thread_2_seq_14171.3755.1151248181
```
历史Archive几乎都在里面，将以前的备份集信息重新导入到当前控制文件中：
```
[root@oracledb2 ~]# su - oracle
[oracle@oracledb2 ~]$ rman target /
RMAN> catalog start with '+DATA/OAORCL/ARCHIVELOG/';
```
&#8195;&#8195;再次执行删除，就可以删除这些历史的Archive了，会提示删除了多少，删除后`lsdg`检查空间。注意保留天数需要与用户进行确认。

## 待补充