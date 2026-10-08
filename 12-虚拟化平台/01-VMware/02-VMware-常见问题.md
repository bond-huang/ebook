# VMware-常见问题

## VMware Workstation问题
### 兼容性问题
Windows10系统，报错描述：
```
VMware Workstation与Device/Credential Guard不兼容。在禁用Device/Credential Guard后，可以运行VMware Workstation
```
我曾经Windows10里面安装了Ubuntu的子系统使用过，处理方法：
- 控制面板-->程序和功能
- 启用或关闭Windows功能
- 关闭如下功能：
    - 适用于Linux的Windows子系统
    - 虚拟机平台
    - Hyper-V（不一定有）
- 重启电脑

官方参考链接：[VMware Workstation and Device/Credential Guard are not compatible" error in VMware Workstation on Windows 10 host (2146361)](http://www.vmware.com/go/turnoff_CG_DG)
### 虚拟化兼容性问题
启动RHEL8时候报错：
```
此平台不支持虚拟化的 AMD-V/RVI。

不使用虚拟化的 AMD-V/RVI，是否继续?
```
在VMware控制台中选择编辑虚拟机：
- 选择并点击`处理器`
- 在虚拟化引擎项中将`虚拟化Intel VT-x/EPT 或AMD-V/RVI(V)`前的勾去掉
- 点击`确认`保存

### 网络问题
虚拟机通过NAT模式连接不了网络，检查：
- 在Windows计算机管理 服务下面查看VMware相关服务是否开启
- 检查Workstation中VMnet8的配置是否正确：
    - VMware Workstation主菜单`编辑`-->虚拟网络编辑器
- VMnet8是否有加入某个网桥，如果有，会有影响

VMnet信息如下：
- 桥接模式（将虚拟机直接连接到外部网络）
- NAT模式（与虚拟机共享主机的IP地址）
- 仅主机模式（在专用网络内连接虚拟机）
    - 如果需要VMware虚拟网卡进行DHCP分配IP给虚拟机的情况下选择此种模式

### 虚拟机开启问题
在用VMware虚拟机的时候，有时打开虚拟机时提示：
```
该虚拟机似乎正在使用中。如果该虚拟机未在使用，请按“获取所有权(T)”按钮获取它的所有权。否则，请按“取消(C)”按钮以防损坏。
```
&#8195;&#8195;两个按钮都解决不了问题，把VMware的进程全部杀掉重新启动程序一样不行，进入到虚拟机的所在的目录下，目录下结尾为`.lck`的文件夹删掉即可，或者备份一份也行，重新打开虚拟机就可以访问了。
## vCenter问题
### VMware Inventory Service服务无法启动
&#8195;&#8195;在Windows系统的vCenter，异常宕机或某些极端情况发生后，启动VMware Inventory Service服务时候报错，vCenter无法使用，报错示例：
```
Windows could not start the VMware Inventory Service on LocaComputer. For more information, review the System Event Log. Ifthis is a non-Microsoft service, contact the service vendor, andrefer to service-specific error code 1.
```
&#8195;&#8195;可能是Inventory Service database问题，数据损坏导致无法启动，需要reset Inventory Service database，对于6.0版本官方参考文档：[How to reset Inventory Service database for vCenter Server 6.0 (2146264)](https://kb.vmware.com/s/article/2146264?lang=en_us)。

## 迁移问题
### 存储迁移问题
#### iso迁移问题
迁移iso文件到其他存储设备上时候，提示：
```
无法访问文件[xxxx xxxx] iso/rhel-server-6.40x86-dvd.iso
```
&#8195;&#8195;原因是iso被一个或多个虚拟机挂载，在系统还挂载文件系统情况下（或者未挂载但未eject），如果直接虚拟机上点击移除，会导致虚拟化hang住无法操作（实际未重启），状态也会异常，并且无法卸载，操作方法：
- 利用RVtools导出即时的vCD清单
- 筛选Connected字段，选择true
- 筛选Device Type字段，选择相应的iso
- 进入挂载iso的每台虚拟机，查看文件系统，如过挂载了iso，检查是否有占用，没有就`umount`
- 然后运行命令`eject`或`eject -v`，两个版本进过测试差异如下：
    - 红帽6版本需要手动在vcenter上卸载光驱
    - 红帽7版本vcenter上虚拟机的光驱已经断开连接

## 待补充