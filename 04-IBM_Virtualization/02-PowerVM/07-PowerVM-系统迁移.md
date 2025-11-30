# PowerVM-系统迁移
官方参考链接：
- [Migrating the Virtual I/O Server by using the viosupgrade command or by using the manual method](https://www.ibm.com/docs/en/power10/9105-41B?topic=migrating-virtual-io-server-by-using-viosupgrade-command)
- [使用 HMC 对移动分区进行迁移](https://www.ibm.com/docs/zh/power9?topic=partition-migrating-mobile-hmc)
- [使用 HMC V 10.2.1040或更低版本迁移移动分区](https://www.ibm.com/docs/zh/power10?topic=partition-migrating-mobile-hmc)

## 迁移问题
官方参考链接：
- [HSCLA219 or HSCLA356 HSCLA29A with OS RC 69 and GPN_FT failure during LPM validation of AIX NPIV Client](https://www.ibm.com/support/pages/hscla219-or-hscla356-hscla29a-os-rc-69-and-gpnft-failure-during-lpm-validation-aix-npiv-client)
- [HSCLA319 during LPM Validation of AIX NPIV Client](https://www.ibm.com/support/pages/hscla319-during-lpm-validation-aix-npiv-client)

### 迁移I/O问题
迁移测试报错：
```
HSCLA340 管理控制台可能无法为目标上所迁移分区的虚拟I/O适配器复制源多路径I/O配置。这意味着出现了下列一种或两种情况:(1)分配给不同源VIOS主机的客户机适配器可能被分配给目标上的单个VIOS主机:(2)分配给单个源VIOS主机的客户机适配器可能被分配给目标上的不同VIOS主机。要复审管理控制台所选择的完整映射列表。请发出命令以列示所迁移分区的虚拟I/O映射。
```
查看详细信息：
```
fscsi2 is not zoned to the same target ports as the source for this client
rc = 69 MIG_LACK_RESOURCE
```
排查过程：
- 一开始怀疑对应vios的光纤线连接的交换机和源端不匹配导致的，调换后报错依旧，实际上没有关系
- 排查了交换机zone，使用wwn号搜索，没有发现异常
- 排查了交换机和线路上的光功率等，无异常
- 查看详细日志，发现只有vios1报错，vios2没有，在验证时候跳过错误，所有fc端口可以通过vios2迁移过去，说明vios1有问题
- 怀疑vios1是克隆或者系统原因导致，重新克隆了一份，也还是不行，打算重新安装但是想想应该不是这个原因
- 最后排查交换机zone，原来是vios1的两个端口绑定了port zone，删掉port zone后即可。新交换机，机器也是新的，线路也是新拉的，没想到有人绑错了zone... ...

## 待补充