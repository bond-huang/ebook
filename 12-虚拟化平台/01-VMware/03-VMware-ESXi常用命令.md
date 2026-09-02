# VMware-ESXi常用命令

## 常用管理命令
### 带外管理
查看带外IP信息：
```sh
esxcli hardware ipmi bmc get
```
如果安装有ipmitool：
```sh
ipmitool lan print 1
```
说明：通道1是绝大多数服务器BMC网口

### 系统管理
版本查看：
```sh
vmware -vl
```
查看ESXi的网络信息：
```sh
esxcli network ip interface ipv4 get
esxcfg-vmknic -l
```
查看软件信息：
```sh
esxcli software vib list
```
查看FC卡信息：
```sh
esxcli storage core adapter list
```
查看存储设备信息：
```sh
esxcli storage core device list
```
列出ESXi NMP多路径下全部SATP声明规则或查询某品牌的：
```sh
esxcli storage nmp satp rule list
esxcli storage nmp satp rule list | grep -i huawei
```
查看ESXi 高级参数：是否对VMFS5/VMFS6存储，使用ATS硬件指令做VMFS心跳更新：
```sh
[root@localhost:~] esxcli system settings advanced list -o /VMFS3/UseATSForHBonVMFS5
   Path: /VMFS3/UseATSForHBOnVMFS5
   Type: integer
   Int Value: 1
   Default Int Value: 1
   Min Value: 0
   Max Value: 1
   String Value: 
   Default String Value: 
   Valid Characters: 
   Description: Use ATS for HB on ATS supported VMFS5 volumes
```
说明：
- -`Int Value=1`（出厂默认）：开启ATS‑Heartbeat，每3秒使用VAAI‑ATS原子指令更新VMFS磁盘心跳块
- `Int Value=0`：关闭ATS心跳，退回到传统普通SCSI读写做磁盘心跳

关闭ATS heartbeat：
```sh
esxcli system settings advanced set -i 0 -o /VMFS3/UseATSForHBOnVMFS5
```
## 待补充
