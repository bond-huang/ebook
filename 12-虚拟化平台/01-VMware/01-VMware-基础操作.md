# VMware-基础操作
个人版用的比较多，企业版用的比较少，一些简单操作记录下来避免忘记。
## 存储操作
### 存储迁移
&#8195;&#8195;VMware平台上一台物理机上有多个虚拟机，一块分配过来的存储盘也经常给多个虚拟机使用。在替换后端存储的时候，可以通过VMware上的虚拟机迁移功能去迁移虚拟机的存储盘（只改变存储盘，不改变虚拟机所在的物理机），优点就是在线迁移，单个虚拟机也很快；缺点就是如果比较多虚拟机太慢了，并发量也需要控制，避免I/O过高产生影响。

具体迁移步骤如下：
- 将新存储跟集群物理机划分好zone，将新磁盘分配给集群，
- 登录VMware的vSphere client，选择对应的集群
- 右键，在`Storage`选择中选择`Rescan Storage`
- 在弹出的“重新扫描存储”对话框中，选择“扫描新的存储设备”，不选中“扫描新的VMFS卷”
- 弹出对话框“新建数据存储”:
    - 类型：选择`VMFS`
    - 名称和设备选择：自定义数据存储名称（例如叫VOLnew）；选择集群中任意一个主机，然后可以看到扫描到的磁盘，选择需要添加的磁盘
    - VMFS版本：有些版本会有此步骤，选择对应的即可
    - 分区配置：默认使用过所有可用分区
    - 即将完成：检查配置，没问题就点击`FINISH`
- 在左侧集群菜单里面可用看到新分配的存储盘：VOLnew
- 点击选择就旧存储盘，例如叫：VOLold，查看存储盘下所有Virtual Machines
- 选中需要迁移得虚拟机，右键，选择`Migrate`（会提示兼容性）：
    - 选择迁移类型：仅更改存储（根据需求选择）
    - 选择存储：磁盘格式选择精简置配；虚拟机存储策略选择保留现有得虚拟机存储策略；勾上禁用此虚拟机得存储DRS;最后选中目标磁盘VOLnew
    - 即将完成：检查配置，没问题就点击`FINISH`
- 然后可以看到精度条，迁移完成后进行检查

注意事项：
- 可以一次选择多个虚拟机进行迁移，但是批量迁移的虚拟机都必须处于同一电源状态
- 批量迁移注意I/O量，一次性过多可能会影响到其它虚拟机正常运行，存储端的性能同样需要关注
- 有些虚拟机可能迁移不了，重启下物理机可能就解决了
- 有些集群开启了自动均衡功能，就是他会自动迁移虚拟机到比较闲的磁盘，可以更改此策略或者对于不打算用的旧盘标记成维护模式
- 确认不要的盘建议核实清楚没有了虚拟机在上面后再删除，最好先标记成维护模式（很多卷名称可能差不多，标记成维护模式的图标会发生改变，区别正在使用磁盘比较明显），过后再删除

### 磁盘扩容
#### 外部存储磁盘扩容
虚拟机上磁盘后端是IBM SVC磁盘，扩容步骤如下：
- 首先找到需要扩容磁盘的ID，进入SVC，对对应的磁盘进行扩展，例如从60G增加到100G：
  - 注意：如果SVC上卷有远程拷贝关系，需要删除关系，再扩容，再建立关系
  - 需要一点时间，耐心等待完成
- 登录vCenter，找到磁盘所在的宿主机：
  - 右键，指向`存储`选项，点击`重新扫描存储`
  - `扫描新的存储设备`和`扫描新的VMFS卷`都默认勾上
  - 点击确定，等待完成扫描
- 登录进入宿主机的ESXI：
  - 点击`存储`选项
  - 找到需要扩容的数据存储设备，右键，点击`增加容量`
  - 选择创建类型：展开现有VMFS数据存储数据区
  - 选择设备：选择需要扩容的设备，点击下一页
  - 选择分区选项：点击编辑一个分区之前所示的卷图像，出现一个进度条，拖动到最右边，点击下一页
  - 即将完成：检查确认无误后点击`完成`

## 虚拟机操作
### 虚拟机删除
### SVC存储磁盘虚拟机清理
需要收集确认的信息：
- 虚拟机IP:192.168.100.110
- 虚拟机所在物理主机的IP：192.168.100.210
- 虚拟机名称：snap_test
- 虚拟机磁盘名称：snap-282b8f27-snap_test_Pool_5T
- VMwate上查看磁盘ID:IBM Fibre Channel Disk (naa.600507680180875b2000000000000462)
    - 需要与SVC上一直
- VMware上LUN ID 62
- SVC上卷名称：VMware_snap_test_Pool_5T
- SVC上映射关系：Lenovo1_vmeare

步骤如下：
- 首先关闭虚拟机
- 虚拟机删除磁盘：操作-编辑设置--删除磁盘--点击确定
- VMcenter上找到磁盘：选择虚拟机，右键：从磁盘删除，点击确认
- VMcenter上找到磁盘：操作--卸载数据存储--弹出对话框选择需要卸载的主机--点击确认

主机上分离磁盘：
- 主菜单选择配置--存储展开选择存储设备：找到对于LUN，选中后点击上面 分离选项，点击确认，查看状态是否分离
- SVC取消映射：找到卷，右键--取消映射所有主机（或者指定主机）
- 重新扫描磁盘：找到主机，右键，选中存储--重新扫描存储。查看删掉存储是否已经无法识别
- 删除虚拟机：卸载数据存储之后，这个虚拟机就没有了

### 克隆虚拟机
#### 克隆一个虚拟机
将一台虚拟机进行克隆步骤（vSphere Client 6.5.0.35000）：
- 找到需要克隆的虚拟机，例如叫TEMPOS，右键，指向`克隆`选项，选择`克隆到虚拟机`
- `1 选择名称和文件夹`：
  - 输入虚拟机名称，例如叫TEMPOS-CLONE
  - 为虚拟机选择位置，选择指定位置
- `2 选择计算资源`：选择指定计算资源，下方`兼容性`对话框会显示兼容性检查结果
- `3 选择存储`：选择指定可用的存储资源，下方`兼容性`对话框会显示兼容性检查结果
- `4 选择克隆选项`：
  - 自定义操作系统：克隆就是想克隆一个一样的，自定义看需求
  - 自定义虚拟机硬件：建议自定义，CPU，内存，磁盘和网络进行修改
  - 创建后打开虚拟机电源：不建议勾上，如果原虚拟机有业务运行，克隆机器网络和前者一样，会产生冲突
- `5 自定义硬件`：自定义CPU，内存，磁盘和网络等配置，网络如果要避免冲突一定不要启动或者直接删掉或者配置其它网络
- `6 即将完成`：核对配置，点击`FINISH`完成配置
- 任务列表中查看任务进度，等待完成

### 虚拟机模板
#### 部署OVF模板
从一个OVA文件导入（vSphere Client 6.5.0.35000）：
- 在左边宿主机列表处右键，选择`部署OVF模板`选项，进入`部署OVF模板`流程
- `1 选择OVF模板`：
  - `URL`：可以是网络地址，或者可以访问的磁盘或DVD等
  - `本地文件`：当前访问vCenter的终端，点击`选择文件`，选择需要导入的OVA文件
- `2 选择名称和文件夹`：
  - 输入虚拟机名称，例如叫TEMPOS
  - 为虚拟机选择指定位置
- `3 选择计算资源`：选择指定计算资源，下方`兼容性`对话框会显示兼容性检查结果
- `4 查看详细信息`：验证模板详细信息，包含发布者，下载大小和磁盘大小，注意：
  - 磁盘大小显示亮相，精简置备和厚置备，注意如果使用厚置备，确保存储空间足够，当然精简置备也需要确认
- `5 选择存储`：选择指定可用的并且容量足够的存储资源，下方`兼容性`对话框会显示兼容性检查结果
- `6 即将完成`：核对配置，点击`FINISH`完成配置
- 任务列表中查看任务进度，等待完成
- 完成后是个虚拟机，可以开机使用，或者直接转成模板

#### 转换成模板
将一个虚拟机转换成模板（vSphere Client 6.5.0.35000）：
- 将虚拟机关机，选中虚拟机，右键，指向`模板`选项，选择`转换成模板`选项
- 任务列表中查看任务进度，等待完成
- 完成虚拟机将不存在了，变成了模板

#### 克隆为模板
将一个虚拟机克隆为模板（vSphere Client 6.5.0.35000）：
- 选中虚拟机，右键，指向`克隆`选项，选择`克隆成模板`选项
- `1 选择名称和文件夹`：
  - 输入虚拟机模板的名称，例如叫TEMPOS
  - 为虚拟机模板选择指定位置
- `2 选择计算资源`：选择指定计算资源，下方`兼容性`对话框会显示兼容性检查结果
- `3 选择存储`：选择指定可用的并且容量足够的存储资源，下方`兼容性`对话框会显示兼容性检查结果
- `4 即将完成`：核对配置，点击`FINISH`完成配置
- 任务列表中查看任务进度，等待完成
- 完成后原虚拟机还在，将生成一个新的模板

### 新建虚拟机
#### 从模板新建
从模板新建一个虚拟机（vSphere Client 6.5.0.35000）：
- 在左边宿主机列表处，右键，选择`新建虚拟机`选项
- `1 选择创建类型`：选择`从模板部署`
- `2 选择模板`：找到需要的模板并选中，点击`NEXT`
- `3 选择名称和文件夹`：
  - 输入虚拟机名称，例如叫TEMPOS
  - 为虚拟机选择位置，选择指定位置
- `4 选择计算资源`：选择指定计算资源，下方`兼容性`对话框会显示兼容性检查结果
- `5 选择存储`：选择指定可用的并且容量足够的存储资源，下方`兼容性`对话框会显示兼容性检查结果
- `6 选择克隆选项`：
  - 自定义操作系统：克隆就是想克隆一个一样的，自定义看需求
  - 自定义虚拟机硬件：建议自定义，CPU，内存，磁盘和网络进行修改
  - 创建后打开虚拟机电源：不建议勾上，如果原虚拟机有业务运行，克隆机器网络和前者一样，会产生冲突
- `7 自定义硬件`：自定义CPU，内存，磁盘和网络等配置，网络如果要避免冲突一定不要启动或者直接删掉或者配置其它网络
- `8 即将完成`：核对配置，点击`FINISH`完成配置
- 任务列表中查看任务进度，等待完成

### 虚拟机磁盘扩容
#### Windows虚拟机扩容C盘
Windows Server 2008 R2版本，只有一个C盘，步骤：
- 找到需要克隆的虚拟机，右键，选择`编辑设置`选项
- 在硬盘处直接添加需要扩容到的容量，例如250G到500G
- 点击`确定`进行修改
- 登录到Windows系统，进入`服务器管理器`
- 展开`存储`选项，点击`磁盘管理`
- 目前还看不到新添加的空间，点击菜单栏的`刷新`，可以看到250G的未分配空间
- 选中需要扩展的磁盘，右键选择`扩展卷`，进入扩展卷向导
- 将可用的250G添加到右边已选的框，点击`下一步`
- 显示已经选择的设置，检查是否正确，点击`完成`确认扩展
- 检查是否正常

注意事项：
- Windows下扩展卷需要连续的扇区，如果多块磁盘需要扩展，根据实际情况再研究扩容方法
- 注意操作前虚拟机做快照或者备份应用数据
- 刚做完快照后，编辑虚拟机磁盘大小时候是灰色的，无法修改磁盘大小，删掉快照就可以了

### 虚拟机挂载CD/DVD
#### 挂着ISO镜像
虚拟机需要一个ISO镜像进行升级，步骤：
- 登录到虚拟机所在宿主机的ESXI，左侧展开`存储`选项
- 选中需要上传到的存储设备，点击`数据存储浏览器`，弹出`数据存储浏览器`对话框：
  - 左侧点击选择需要上传到的存储设备
  - 点击`上载`，从本机上传IOS文件，查看下面任务进度，等待上传完成，成功后在右侧显示
- 上传成功后，找到需要操作的虚拟机，展开`操作`选项，点击`编辑设置`：
  - 在`CD/DVD驱动器`那选择`数据存储ISO文件`，弹出相应对话框：
    - 数据存储：选择对应数据存储
    - 内容：选择需要的ISO文件
    - 信息：显示ISO文件的信息
  - 右边`已连接`勾上
  - 点击`确认`
- 可以去系统使用CD/DVD驱动器了，镜像也挂载了

## 待补充