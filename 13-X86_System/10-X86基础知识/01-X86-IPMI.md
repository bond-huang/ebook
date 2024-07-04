# X86-IPMI
学习参考：
- AI回答
- [深入理解ipmitool：揭秘BMC与IPMI的智能服务器管理(带外管理)](https://cloud.tencent.com/developer/article/2375158)

## IPMI介绍
&#8195;&#8195;Intelligent Platform Management Interface(IPMI智能平台管理接口),是一种独立于操作系统的标准接口，用于远程管理和监控计算机硬件，包括服务器、存储设备等。它允许系统管理员即使在主机操作系统未运行、崩溃或已关机的情况下，也能进行系统管理和监控。
### IPMI的主要功能
IPMI的主要功能如下：
- 远程控制电源：可以通过IPMI远程开机、关机、重启服务器等操作
- 硬件监控：IPMI可以监控服务器的硬件健康状态，（CPU、内存、硬盘、风扇、机框等）的温度、电压等健康状态进行检测。它能够通过传感器读取信息，并在监测到异常时发出警报
- 访问系统日志：IPMI可以访问系统事件日志，记录系统硬件的错误信息和重要事件
- KVM over IP：许多IPMI实现（如iDRAC、ILO）还支持KVM over IP（键盘、视频、鼠标），允许管理员通过网络访问服务器的控制台，就像直接连接到服务器一样
- 虚拟媒体：IPMI可以远程挂载ISO文件或映像文件，让系统看起来像有一个本地的CD/DVD/USB驱动器
- 设备信息管理：记录服务器型号、制造商、日期、各部件生产和技术信息、机箱信息、主板信息等、BMC信息（服务器主机名、IP、BMC固件版本等信息）

### IPMI的组件
IPMI主要由以下几个组件组成：
- BMC (Baseboard Management Controller)：这是IPMI的核心硬件组件，通常嵌入在服务器的主板上。BMC管理并监控传感器数据，处理远程管理命令，并生成系统事件日志
- IPMI Firmware：BMC上运行的固件，提供IPMI指令集的具体实现，并允许远程和本地管理工具与BMC进行交互
- 传感器：用于监控服务器的不同部分，如温度传感器、电压传感器和风扇传感器
- 系统事件日志 (SEL)：BMC记录的系统硬件事件和错误日志
- 管理工具：软硬件工具，用于通过IPMI接口远程管理服务器。这些工具可以是命令行工具（如`ipmitool`）、基于Web的界面（如各厂商的管理控制台）、或者是综合监控和管理平台的一部分。

### 使用IPMI的常见场景
常用IPMI的常见场景：
- 远程重启服务器：当系统挂起或无法通过操作系统正常重启时，使用IPMI可以远程硬重启服务器
- 系统维护：在需要更新固件或进行硬件更换时，使用IPMI可以远程进行各种操作，而不需要通过操作系统
- 硬件故障排查：通过读取传感器数据和系统事件日志，可以探查到硬件问题如过热、硬盘故障、电源问题等
- 操作系统崩溃情况下的管理：即使操作系统崩溃了，管理员仍然可以通过IPMI进行诊断、收集日志并重新启动系统

## IPMI访问
### IPMI密码恢复或重置
&#8195;&#8195;如果忘记了IPMI的密码，恢复或重置密码的方法通常取决于具体的硬件制造商和服务器型号。以下是一些常见的解决方法和步骤。

#### Dell服务器（iDRAC）
对于Dell服务器，iDRAC是其远程管理的实现。可以通过以下方法重置iDRAC密码：
- 使用物理访问通过BIOS设置重置：
    - 重新启动服务器并进入BIOS设置（通常通过按F2键）
    -  进入`iDRAC Settings`
    - 找到`User Configuration`，选择用户并重置密码。
- 使用命令行工具 `racadm`：
    - 如果有远程控制台访问，可以使用命令行工具 `racadm`来重置密码：
    ```sh
    racadm set idrac.users.2.password newpassword
    ```

#### Hewlett Packard Enterprise服务器（iLO）
对于HPE服务器，可以通过以下方法重置iLO密码：
- 使用物理访问通过BIOS设置重置：
    - 重新启动服务器并进入BIOS设置（通常通过按F9键）
    - 进入`System Utilities` > `iLO 4 Configuration Utility`
    - 找到用户设置并重置密码
- 使用 `HPONCFG` 命令行工具：
    ```sh
    hponcfg -w output.xml
    # 编辑 output.xml 文件，设置新密码
    hponcfg -f output.xml
    ```

#### Lenovo(IBM)服务器（IMM）
对于Lenovo服务器，可以通过以下方法重置IMM密码：
- 使用物理访问通过BIOS设置重置：
    - 重新启动服务器并进入BIOS设置（通常通过按F1键）
    - 进入`System Settings` > `Integrated Management Module`
    - 找到用户设置并重置密码
- 使用命令行工具 `immpaswd`：
    ```sh
    immpaswd -d user -n new_user -p new_password
    ```

#### 使用外部硬件工具
一些服务器支持通过特定硬件接口（如JBOD接口或IPMI的物理接口）进行密码重置。

#### 重置BMC（Baseboard Management Controller）
有些服务器允许通过硬件跳线（jumper）或按钮重置BMC配置，包括IPMI密码。
- 检查服务器的硬件手册，寻找BMC重置跳线或按钮的位置。
- 重启服务器，并按照手册指示进行重置操作

#### 使用操作系统ipmitool
如果可以登录到主机操作系统，并有足够的权限，可以使用IPMI工具进行管理：
```sh
# 列出所有IPMI用户
ipmitool user list

# 重置指定用户的密码
ipmitool user set password USERID new_password
```
## 待补充
