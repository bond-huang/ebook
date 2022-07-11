# CDC-安装与卸载
## IBM i安装与卸载
官方参考链接：
- [Installing the CDC Replication Engine for Db2 for i version 11.4](https://www.ibm.com/docs/en/idr/11.4.0?topic=credi-installing-cdc-replication-engine-db2-i-version-114)
- [11.4.0-Uninstalling the CDC Replication Engine for Db2® for i](https://www.ibm.com/docs/en/idr/11.4.0?topic=i-uninstalling-cdc-replication-engine-db2)

### 安装CDC
#### 系统要求
&#8195;&#8195;CDC要求您分配一个端口以与运行管理控制台和其他服务器的客户端工作站进行通信。这些端口必须可以通过防火墙访问。具体说明如下表：

协议|默认端口|IN/OUT|目的
:---:|:---:|:---:|:---
TCP|11111|IN|接受来自管理控制台、访问服务器、命令行实用程序和其他CDC Replication安装的连接作为复制源
UDP|2222|IN|仅当在Access Server 10.2.1及更早版本中启用了自动发现功能时，才需要此端口。该端口侦听来自访问服务器的广播，该服务器检测所有正在运行的CDC Replication安装
TCP|11111|OUT|在管理控制台中添加订阅时，如果订阅使用配置为数据源的数据存储(源数据存储)，那么需要一个TCP端口，以便订阅可以连接到目标数据存储
UDP|10101|OUT|Auto-discovery回复在此端口上发送回Access Server。默认开启Auto-discovery

软硬件、磁盘、内存等要求官方参考链接：[System requirements for CDC Replication Engine for Db2® for i](https://www.ibm.com/docs/en/idr/11.4.0?topic=i-system-requirements-cdc-replication-engine-db2)
#### 用户登录
建议创建专门的用户进行安装，用户要求如下：
- 需要有权限：`*SECADM`,`*JOBCTL`,`*ALLOBJ`,`*SAVSYS`,`*AUDIT`,`*IOSYSCFG`,`*SPLCTL`
- 不要使用`D_MIRROR`用户

例如创建CDCADMIN用户：
```
CRTUSRPRF USRPRF(CDCADMIN) 
          SPCAUT(*SECADM *JOBCTL *ALLOBJ *SAVSYS *AUDIT *IOSYSCFG *SPLCTL)
```
#### 安装文件及程序准备
将安装文件通过FTP等传输到系统指定库，例如叫`IBMTEMP`，然后使用下面命令恢复安装程序：
```
RSTOBJ OBJ(DMCINSTALL) SAVLIB(V11R4M0) 
       DEV(*SAVF) SAVF(IBMTEMP/V11R4M0) RSTLIB(QTEMP)
```
#### 运行安装程序
运行如下命令开启安装程序：
```
?QTEMP/DMCINSTALL
```
按`F4`进入安装提示屏幕，在选项中输入相关信息：
- `Device Name`：用于运行安装程序的Savefile：`*SAVF`
- `Savefile Name for Product Library Name`：`V11R4M0`，对应`Library Name`为存放此Savefile的库，即`IBMTEMP`
- `Savefile Name for Tutorial Name`：`V11R4M0TUT`，对应`Library Name`为存放此Savefile的库，即`IBMTEMP`，还可以输入`*LIBL`以指定列表中的库集

示例如下：
```
                       Product Installation (DMCINSTALL)     
Type choices, press Enter.                                    
                                                              
Device name  . . . . . . . . . .   *SAVF         *SAVF        
Savefile name for product  . . .   V11R4M0       Name         
  Library Name . . . . . . . . .     IBMTEMP     Name, *LIBL  
Savefile name for tutorial . . .   V11R4M0TUT    Name         
  Library Name . . . . . . . . .     IBMTEMP     Name, *LIBL  
```
对应命令如下：
```
QTEMP/DMCINSTALL SAVFPROD(IBMTEMP/V11R4M0) 
                 SAVFTUTOR(IBMTEMP/V11R4M0TUT)
```
按`Enter`继续。
#### 用户Profile选择
&#8195;&#8195;上一步执行后，将出现`D_MIRROR User Profile Exists`的提示屏幕。表明安装程序检测到存在`D_MIRROR`用户配置文件。此用户配置文件可能存在于先前安装的CDC Replication中。CDC Replication要求为版本11.4正确配置`D_MIRROR`用户配置文件：
- `F2`：保持`D_MIRROR`用户配置文件不变。如果用户配置文件是在安装以前的CDC Replication版本期间创建的，并且其配置未更改，则用户配置文件对于版本11.4应该仍然有效
- `F3`：取消安装
- `F4`：重新配置`D_MIRROR`用户配置文件

验证用户配置文件是否配置正确：
```
DSPUSRPRF USRPRF(D_MIRROR)
```
验证以下参数：
- Special authority：`*JOBCTL`
- Password is *NONE：`*YES`(密码的使用是可选的)
- Set password to expired：`*NO`
- Message queue：`QUSRSYS/D_MIRROR`

按`Enter`继续。
#### 许可协议
屏幕上显示的软件许可协议中的条款，按`F2`接受并开始安装。然后按`Enter`继续。
#### 指定产品和教程库
在`Product and Tutorial Library`屏幕中输入下面信息：
- `Product Library`：要安装CDC Replication的库的名称。默认是`DMIRROR`
- `Tutorial Library`：要安装教程表的库的名称（可选），默认是`DTUTOR`
    - 必须将CDC Replication和教程表安装到不同的库中
- `Product IASP Device`：要安装CDC Replication的IASP设备的名称：
    - 如果要在IASP上安装CDC Replication，请输入IASP设备的名称
    - 如果没有ASP设备，使用默认值`*SYSBAS`

如果指定了IASP设备的名称，则会出现安装类型屏幕：
- `F`(Full product installation)：将CDC Replication产品安装到指定的IASP设备中，并将所需的支持对象(`*SBSD``*CLSD`和`*JOBQ`)安装到当前连接到IASP设备的机器的系统ASP上的工作库中
- `W`(Work library only)：仅将所需的支持对象(`*SBSD``*CLSD`和`*JOBQ`)安装到连接到IASP设备的机器的系统ASP上的工作库中：
    - 如果已经在连接的IASP设备上安装了CDC Replication的完整产品，请指定此选项
    - 当实现了可切换的IASP环境并需要在Secondary机器上使用另一个工作库副本时，也可以指定此选项
    - 安装程序会自动创建工作库。工作库的名称是产品库的前八个字符加上01。例如，`DMIRROR01`
    - 如果库存在，则安装程序结束，提示`Work library DMIRROR01 already exists! Installation will terminate.`

如果指定了现有库的名称，则会出现`Existing Library Specified`屏幕：
- 默认产品库是`DMIRROR`
- 在`Replace Specified Library`字段中，指定要安装CDC Replication的库的名称
- 如果要在指定的产品库中安装CDC Replication，请输入`Y`，选择`Y`删除该库的所有现有内容；如果要安装到不同的库中，请输入`N`，并指定不同库的名称

#### 完成安装
&#8195;&#8195;安装程序会创建指定的库并在系统上安装CDC Replication。安装完成后，会生成一条最终消息以指示安装是否成功。如果安装不成功，则会在作业日志中放置错误消息。在再次运行安装程序之前，使用`DSPJOBLOG`命令来识别错误并采取必要的纠正措施。安装成功消息示例：
```
The product installation is complete.See Installation Guide for post installation activities.
```
### 卸载CDC