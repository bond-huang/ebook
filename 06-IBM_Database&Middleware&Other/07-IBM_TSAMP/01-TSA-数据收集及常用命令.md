# TSA-数据收集及常用命令
## 数据收集
Linux系统日志位置：
```
/var/log/messages
```
TSA数据收集官方说明：
- [Collecting data for: Tivoli System Automation for Multiplatforms](https://www.ibm.com/support/pages/collecting-data-tivoli-system-automation-multiplatforms)

## 常用命令
参考链接：
- [TSAMP - Commands cheatsheet for new Administrator ](https://www.ibm.com/support/pages/tsamp-commands-cheatsheet-new-administrator)
- [Commands Reference](https://www.ibm.com/docs/en/tsafm/4.1.1?topic=reference-commands)

### 查看命令
查看正在运行TSA版本：
```sh
samversion
lssamctrl
lssrc -ls IBM.RecoveryRM
lsrsrc -Ab -c IBM.CHARMControl
```
查看集群状态情况：
```sh
lssam
```
显示SAM控制信息：
```sh
lssamctrl
```
查看当前`IBM.RecoveryRM` daemon的"master"节点:
```sh
lssamctrl -V | grep -i master
```
查看domain的当前状态以及已配置的domin：
```sh
lsrpdomain
```
域处于联机状态下，查看群集中可用的节点及其状态：
```sh
lsrpnode
```
列出已定义的通信组：
```sh
lscomg
```
列出所有定义的关系及其持久属性和动态属性：
```sh
lsrel -Ab
```
列出资源组或资源组成员创建请求：
```sh
lsrgreq -L
```
列出资源组成员请求：
```sh
lsrgreq -L -m
```
要列出活动的 tiebreaker资源以及CritRsrcProtMethod的值：
```sh
lsrsrc -Ab -c IBM.PeerNode
```
### 启停命令
#### 启停domain
启动domian：
```sh
startrpdomain <DOMAIN_NAME>
```
停止domian：
```sh
stoprpdomain <DOMAIN_NAME>
```
&#8195;&#8195;如需在不首先使所有资源脱机的情况下强制停止域，先将`CritRsrcProtMethod`更改为5，以确保不会因守护进程异常退出而重新启动：
```sh
chrsrc -c IBM.PeerNode CritRsrcProtMethod=5
stoprpdomain -f <DOMAIN_NAME>
```
#### 资源启停
关闭所有资源组：
```sh
chrg -o offline -s 1=1
```
启动所有资源组：
```sh
chrg -o online -s 1=1
```
启动某个资源组：
```sh
chrg -o online <RG name>
```
停止某个资源组：
```sh
chrg -o offline <RG name>
```
停止资源组中某个应用：
```sh
stoprsrc -s 'Name=="<app_name>"' IBM.Application
```
重置资源组中某个应用：
```sh
resetrsrc -s 'Name=="<app_name>"' IBM.Application
```
### 资源请求
提交请求：
```sh
rgreq -o <request_type> <RG or RG member name>
```
锁定所有资源组：
```sh
rgreq -o lock -s 1=1
```
解锁所有资源组：
```sh
rgreq -o unlock -s 1=1
```
切换资源组：
```sh
rgreq -o move <RG name>
```
请求操作类型：
- `start`：启动资源组的请求
- `stop`：停止资源组的请求
- `move`：资源组及其成员移动到群集中的其他节点
- `cancel`：取消以前输入的请求
- `movecancel`：取消先前输入的移动请求
- `lock`：锁定资源组，锁定后保持当前状态不自动化切换
- `unlock`：解锁资源组

### 集群配置
创建集群必须两个步骤：
```sh
preprpnode <node01> <node02> <node03>...
mkrpdomain <domain_name> <node01> <node02> <node03>...
```
从节点/服务器中完全删除所有群集配置（节点会重启）：
```sh
/usr/sbin/rsct/install/bin/recfgct
```
### 配置调整
要调用手动模式：
```sh
samctrl -M T  ## TSAMP Control Manual Mode is True
```
关闭手动模式：
```sh
samctrl -M F  ## TSAMP Control Manual Mode is False
```
## 待补充