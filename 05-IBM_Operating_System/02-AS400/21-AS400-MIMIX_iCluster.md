# AS400-MIMIX_iCluster
## iCluster
### 常用命令
#### Nodes
命令|描述
:---|:---
DMADDNODE|Add a cluster node
DMCHGNODE|Change a cluster node
DMRMVNODE|Remove a cluster node
DMDSPNODE|Display a cluster node
DMWRKNODE|Display the Work With Nodes screen

#### Groups
命令|描述
:---|:---
DMADDGRP|Add a group
DMCHGGRP|Change a group
DMRMVGRP|Remove a group
DMADDBACK|Add a node to a recovery domain
DMRMVBACK|Remove a node from a recovery domain
DMDSPGRP|Display a group
DMWRKGRP|Display the Work With Groups screen

#### Cluster operations
命令|描述
:---|:---
DMSTRNODE|Start cluster operations at a node
DMENDNODE|End cluster operations at a node
DMREJOIN|Start cluster operations at this node
DMSTRGRP|Start cluster operations for a group
DMENDGRP|End cluster operations for a group
DMSTRSWO|Start switchover for a group
DMCHGROLE|Change a group's primary node

#### Status and history monitor
WRKHASMON|Work with the iCluster primary status monitor
WRKHATMON|Work with the iCluster backup status monitor
DSPHASMON|Display the iCluster primary status monitor
CHGHASMON|Change the iCluster primary history monitor
PRGHASMON|Purge the iCluster primary monitor history
WRKCSMON|Work with the full cluster status monitor

#### Journal management and analysis                                             
DMWRKJRN|Work with journals for iCluster
CHGHAJRN|iCluster change journal receiver
DLTHAJRCV|Delete journal receivers
ENDHADJRCV|End journal management job
DMDSPJNMNG|Display journal management details
DMRMVJNMNG|Remove journal management entry
DMSTRJNMNG|Restart journal management jobs
DMANZJRN|Analyze journals

## MIMIX