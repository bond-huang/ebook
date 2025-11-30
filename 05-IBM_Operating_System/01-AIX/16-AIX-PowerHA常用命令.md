# AIX-PowerHA常用命令

## HA常用命令
### 检查命令
常用检查命令：
```
lscluster -m
lssrc -g cluster
lssrc -s clcomd
lssrc -ls clstrmgrES
lsrpnode
lsrpdomain
/opt/rsct/bin/caa_config -s <domain>
/usr/sbin/rsct/bin/caa_config -s  <domain>
/usr/es/sbin/cluster/utilities/clRGinfo -p
/usr/es/sbin/cluster/utilities/cltopinfo -i
```

## 待补充