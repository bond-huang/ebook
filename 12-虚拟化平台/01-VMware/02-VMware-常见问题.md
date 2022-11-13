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
## 待补充