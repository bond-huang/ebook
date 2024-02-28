# HMC-常用配置
## 网络配置
### 防火墙
hmc防火墙信息：[HMC Firewall Information](https://www.ibm.com/support/pages/hmc-firewall-information)
## HMC间互相认证
### 两台HMC互相认证配置
如果未配置，进行跨HMC分区迁移时候，报错如下：
```
HSCL3653 未对远程用户 hscroot 正确设置源管理控制台与目标管理控制台之间的 Secure Shell (SSH) 通信配置。请运行 mkauthkeys 命令以设置 SSH 通信认证密钥。
```
进行免密测试，不通过示例如下：
```
hscroot@HMC666:~> mkauthkeys --ip 100.26.17.82 -u hscroot --test
HSCL3653 The Secure Shell (SSH) communication configuration between the source and destination management consoles has not been set up properly for the remote user hscroot. Run the mkauthkeys command to set up the SSH communication authentication keys
```
配置示例如下：
```
hscroot@HMC666:~> mkauthkeys --ip 100.26.17.82 -u hscroot
Enter the password for user hscroot on the remote host 100.26.17.82:
hscroot@HMC666:~> mkauthkeys --ip 100.26.17.82 -u hscroot --test
hscroot@HMC666:~> 
```
然后在另一端同样进行以上配置，就可以互相免密进行系统迁移操作了。HMC免密配置mkauthkeys命令：[HMC Manual Reference Pages  -MKAUTHKEYS (1)](https://www.ibm.com/docs/en/power8?topic=commands-mkauthkeys)

## 待补充