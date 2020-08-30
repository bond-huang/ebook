# SystemX-ASU使用
在System X系列服务器中是一个很强大很有用的工具，目前主要接触用来改序列号等操作。
### ASU基本使用
##### 下载及说明
IBM介绍和下载地址：[https://www.ibm.com/support/pages/node/811842](https://www.ibm.com/support/pages/node/811842)

Lenovo介绍和下载地址：[https://support.lenovo.com/gb/en/solutions/lnvo-asu](https://support.lenovo.com/gb/en/solutions/lnvo-asu)

IBM官方使用说明：[https://www.ibm.com/support/pages/node/823076](https://www.ibm.com/support/pages/node/823076)

##### 使用方法
&#8195;&#8195;根据自己客户端类型下载对应软件包，一般是Windows系统，使用客户端（个人笔记本或者其它终端设备）连接至X系列服务器的IMM管理端口，保证终端和服务器之间网络是连通的（如果是直连配置同一网段IP即可），在客户端打开CMD 窗口，进入ASU 所在目录，即可进行操作。

##### 查看服务器基本信息
按照以上方法连接到服务器后，使用命令如下命令：
```
D:\ASU\asu show --host 192.168.70.125
```
说明：
- 示例中是直接把ASU软件所在的目录ASU目录放在D盘根目录下
- X系列服务器IMM口默认IP一般是192.168.70.125
- 能够连接上回车后会显示很多设备信息

##### 更改服务器的序列号
在更换完主板后，通常需要进行更改，使用命令如下命令更改服务器的型号：
```
D:\ASU\asu sed SYSTEM_PROD_DATA.SysInfoSerialNum 7835HNY --host 192.168.70.125
```
说明：
- 其实就是更改在asu show 中查看到的SYSTEM_PROD_DATA.SysInfoSerialNum属性的值
- 注意序列号是7位字母和数字的组合
- 修改该完成后通过asu show查看确认

##### 更改服务器的型号
上面能正常查看服务器基本信息后，使用命令如下命令更改服务器的型号：
```
D:\ASU\asu sed SYSTEM_PROD_DATA.SysInfoName 6241GAC--host 192.168.70.125
```
说明：
- 其实就是更改在asu show中查看到的SYSTEM_PROD_DATA.SysInfoName属性的值
- 6241GAC是服务器型号，对应就是X3950 X6
- 修改该完成后通过asu show查看确认

##### 更改服务器的固定资产号
在更换完主板后，通常需要进行更改，使用命令如下命令更改服务器的型号：
```
D:\ASU\asu sed SYSTEM_PROD_DATA.SysEncloseAssetTag tysv08110042 --host 192.168.70.125
```
说明：
- 其实就是更改在asu show中查看到的SYSTEM_PROD_DATA.SysEncloseAssetTag属性的值
- 注意AssetTag的格式，可以参照修改签asu show查看到的
- 修改该完成后通过asu show查看确认
