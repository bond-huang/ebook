# vCenter-故障处理
## Inventory Service database
### 重置Inventory Service database
&#8195;&#8195;在Windows系统的vCenter，异常宕机或某些极端情况发生后，启动VMware Inventory Service服务时候报错，vCenter无法使用，需要重置Inventory Service database。操作的是vCenter Server 6.0版本，注意事项：
- 重置Inventory Service database的操作是破坏性操作；
- 如果重置Inventory Service database：
    - vCloud Director和vCenter Server将不同步，并且没有关于受管虚拟机的准确信息；
    - 正在通过vSphere Profile-Driven Storage Service使用的任何存储配置文件都将丢失；
    - 所有对象标记将在重置过程中被删除
- 操作前创建vCenter Server的快照或备份

官网下载的文件：
- 2119422_reset-isdb6.0-bundle.zip：手动操作下载此文件
- 2119422_6.0-ISDB-Reset-PS1.zip：自动重置脚本

使用路径说明（具体路径根据实际情况），官方文档使用的变量，对应我操作系统文件夹：
- `%PROGRAMFILES%`,路径`C:\Program Files`
- `%PROGRAMDATA%`,路径：`C:\ProgramData`，注意此文件夹是个隐藏文件夹

&#8195;&#8195;上面是官方文档描述的风险，但是时间搞完后，受管的ESXI和存储设备等信息都在，也没发现丢失啥配置。根据文档步骤一步一步进行的，当然可以直接运行文件`2119422_6.0-ISDB-Reset-PS1.zip`解压后的PowerShell脚本。文档及相关文件下载地址：[How to reset Inventory Service database for vCenter Server 6.0 (2146264)](https://kb.vmware.com/s/article/2146264?lang=en_us)。
#### 创建全新的Inventory Service database
停止服务：
- VMware vSphere Web Client
    - VMware Performance Charts
- VMware VirtualCenter Server
    - VMware vService Manager
    - VMware vCenter workflow manager
    - VMware Syslog Collector
    - VMware vSphere Profile-Driven Storage Service
    - VMware Content Library Service
    - VMware ESX Agent Manager
- VMware Inventory Service

进入`C:\Program Files\VMware\vCenter Server\invsc\scripts`文件夹：
- 备份`createdb.bat`和 exec.bat`文件
- 将`2119422_reset-isdb6.0-bundle.zip`解压，将里面文件`createdb.bat`和`exec.bat`复制到`C:\Program Files\VMware\vCenter Server\invsc\scripts`文件夹

进入`C:\Program Files\VMware\vCenter Server\invsc\lib\server\config`文件夹：
- 打开`dataservice.properties`文件
- 查找`dataservice.xdb.password`条目，删除所有除`@`外的特殊字符：
    - 例如`dataservice.xdb.password=cSIOMV*)2>=$EGvg`，删除后是`dataservice.xdb.password=cSIOMV2EGvg`
- 查找`dataservice.xdb.dir`条目，并将`\`的更改为`/`：
    - 例如`dataservice.xdb.dir=C:\ProgramData\VMware\vCenterServer\data\invsvc\xdb`修改后是`dataservice.xdb.dir=C:/ProgramData/VMware/vCenterServer/data/invsvc/xdb`
- 不要关闭此文件的编辑，待会继续使用

进入``C:\Program Files\VMware\vCenter Server\vpxd\endpoints`文件夹：
- 打开`qs-endpoint.xml`文件
- 查找`<instanceUuid>`，例如：`<instanceUuid>1c9f5ff7-c844-45d1-bf62-a4e9c0860a57</instanceUuid>`
- 复制`<instanceUuid>`与`</instanceUuid>`之间的内容，例如：`1c9f5ff7-c844-45d1-bf62-a4e9c0860a57`

返回到`dataservice.properties`文件编辑：
- 创建`dataservice.instanceuuid`条目并添加上一步复制的UUID，条目示例：
    - `dataservice.instanceuuid=1c9f5ff7-c844-45d1-bf62-a4e9c0860a57`
- 保存并关闭`dataservice.properties`文件

进入`C:\ProgramData\VMware\vCenterServer\data\invsvc`目录：
- 将里面的`xdb`文件夹重命名为`xdb.old`

进入到CMD命令提示符窗口：
- `cd`到目录`C:\Program Files\VMware\vCenter Server\invsc\scripts`
- 执行`createdb.bat`脚本：
    - 正常最后一条应该是：`Store 1c9f5ff7-c844-45d1-bf62-a4e9c0860a57 has been shutdown`
    - 如果运行失败，最后是：`EXECUTION FAILED`

#### 向Component Manager重新注册Inventory Service
进入`C:\Program Files\VMware\vCenter Server\vpxd\inventoryservice-registration`文件夹：
- 备份三个文件（不存在的不管）：`register-is.bat`，`vcregtool.bat`，`GetAppDataDir.vbs`
- 将`2119422_ reset-isdb6.0-bundle.zip`中解压的`register-is.bat`，`vcregtool.bat`和`GetAppDataDir.vbs`三个文件复制当前目录

进入`C:\ProgramData\VMware\vCenterServer\`文件夹：
- 创建文件夹`ssl`
- 进入CMD，`cd`到目录`C:\Program Files\vmware\vcenter server\vmafdd\`，执行命令：
    - `vecs-cli.exe entry getkey --store vpxd --alias vpxd --output C:\ProgramData\VMware\vCenterServer\ssl\rui.key`
    - `vecs-cli.exe entry getcert --store vpxd --alias vpxd --output C:\ProgramData\VMware\vCenterServer\ssl\rui.crt`
    
按所列顺序启动以下服务：
- VMware Inventory Service
- VMware VirtualCenter Server

进入`C:\Program Files\VMware\vCenter Server\invsc\lib\server\config`文件夹：
- 打开`dataservice.properties`文件
- 查找条目`dataservice.cm.url=`，例如：
    - `dataservice.cm.url=http://localhost:18090/cm/sdk/?hostid=d820e6b0-c730-11e4-bddf-005056031f43`
- 复制后面url，例如：`http://localhost:18090/cm/sdk/?hostid=d820e6b0-c730-11e4-bddf-005056031f43`
- 进入CMD，`cd`到目录`C:\Program Files\VMware\vCenter Server\vpxd\inventoryservice-registration`
- 运行命令（注意刚才复制的url）：`register-is.bat "http://localhost:18090/cm/sdk/?hostid=d820e6b0-c730-11e4-bddf-005056031f43" http://localhost:8095`

按顺序启动以下服务：
- VMware ESX Agent Manager
- VMware Content Library Service
- VMware vSphere Profile-Driven Storage Service
- VMware Syslog Collector
- VMware vCenter workflow manager
- VMware vService Manager
- VMware vSphere Web Client
- VMware Performance Charts

&#8195;&#8195;服务全部启动后，可以使用web进入vCenter，用户密码还是之前的，里面的esxi机器及存储应该都还在。注意官方文档里面步骤一步都不能少，都得做。

## 虚拟机问题
### 虚拟机宕机无法启动
#### 系统磁盘满导致无法启动
启动虚拟机报错示例：
```
当前时间无法允许该操作，因为该虚拟机有一个未解决得问题：“msg.hbacommon:There is no more space for virtual disk ......
```
虚拟机上提示：
```
没有更多空间可供虚拟磁盘“1.vmdk”使用，也许可以通过释放相关卷上得磁盘空间并单击重试 继续此会话
```
现象及可能原因：
- 检查系统所在磁盘，可以看到可用空间为0，可能存在超分的情况
- 系统启动后，进入系统检查，文件系统使用率不一定会很高，不是文件系统爆满导致的

解决建议：
- 系统所在的磁盘进行扩容
- 对于超分的磁盘，注意使用情况

## 待补充
