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
- 仅主机模式（再专用网络内连接虚拟机）

## 待补充