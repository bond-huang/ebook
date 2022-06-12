# RHEL-分析服务器及获取支持
## 分析和管理远程服务器
### Web控制台描述
&#8195;&#8195;Web控制台是适用于RHEL8 的基于Web型管理界面，专为管理和监控您的服务器而设计。它的基础是开源Cockpit服务。可以使用 Web 控制台监控系统日志并查看系统性能图表。
### 启用Web控制台
&#8195;&#8195;除了最小安装外，RHEL8的所有安装版本中都默认安装Web控制台。可以使用`yum`命令安装Web控制台：
```
[root@redhat8 ~]# yum install cockpit
```
&#8195;&#8195;启用并启动`cockpit.socket`服务，它会运行一个Web服务器。如果需要通过Web界面连接系统，必须执行这个步骤：
```
[root@redhat8 ~]# systemctl enable --now cockpit.socket
Created symlink /etc/systemd/system/sockets.target.wants/cockpit.socket → /usr/lib/
systemd/system/cockpit.socket.
```
&#8195;&#8195;如果使用的是自定义防火墙配置集，需要将`cockpit`服务添加到`firewalld`，以在防火墙中开启端口`9090`：
```
[root@redhat8 ~]# firewall-cmd --add-service=cockpit --permanent
Warning: ALREADY_ENABLED: cockpit
success
[root@redhat8 ~]# firewall-cmd --reload
success
```
### 登录Web控制台
&#8195;&#8195;Web控制台提供自己的Web服务器。可以使用系统上任何本地帐户的用户名和密码登录，包括root 用户在内。在Web浏览器中打开地址：
```
https://servername:9090
```
说明：
- 其中`servername`是服务器的主机名或IP地址
- 连接将受到TLS会话的保护。默认情况下，系统安装有一个自签名的 TLS 证书，首次连接时，Web浏览器可能会显示安全警告。
- 在登录屏幕中输入用户名和密码
- 选择`Reuse my password for privileged tasks`选项。这将允许使用`sudo`特权执行命令，从而执行诸如修改系统信息或配置新帐户之类的任务
- 登录后Web控制台在标题栏右侧显示用户名。如果选择了`Reuse my password for privileged tasks`选项，用户名的左侧会显示`Privileged`图标
- 如果以某个非特权用户身份登录，则不显示`Privileged`图标

### 更改密码
&#8195;&#8195;特权和非特权用户都可在登录Web控制台期间更改自己的密码。单击导航栏中的`Accounts`。单击帐户标签，以打开帐户详细信息页面：
- 作为非特权用户时，只能设置或重置自己的密码，或者更改SSH公钥
- 若要设置或重置密码，单击`Set Password`

### 使用Web控制台进行故障排除
&#8195;&#8195;Web控制台是一个功能强大的故障排除工具。可以实时监控基本系统统计信息，检查系统日志，并快速切换到Web控制台中的终端会话，以便从命令行界面中收集其他信息。
#### 实时监控系统统计信息
&#8195;&#8195;单击导航栏中的`Overview`来查看系统的相关信息，如硬件类型、操作系统和主机名等。如果以非特权用户身份登录，可以查看所有信息，但不能修改值：
- 单击`Overview`页面上的`View graphs`，查看列出CPU活动、内存用量、磁盘 I/O 和网络利用率等当前系统性能的图表

#### 检查和过滤Syslog事件
通过导航栏中的`Logs`，可以访问系统日志分析工具：
- 可以使用页面上的菜单，根据日志记录日期范围或严重性级别来过滤日志消息
- Web控制台使用当前日期作为默认日期，可以单击日期菜单并指定任何日期范围
- `Severity`菜单提供了从`Everything`到更具体严重性条件（如Alert and above和Debug and above等）等不同范围的选项

#### 从终端会话运行命令
&#8195;&#8195;通过导航栏中的`Terminal`，可以在Web控制台界面内访问全功能终端会话。这样就能运行任意命令来管理和操作系统，还能执行Web控制台提供的其他工具所不支持的任务。
#### 创建诊断报告
&#8195;&#8195;诊断报告中汇集了来自RHEL系统的配置详情、系统信息和诊断信息。已完成报告中收集的数据包括可用于对问题进行故障排除的系统日志和调试信息： 
- 以特权用户身份登录Web控制台
- 单击导航栏中的`Diagnostic Reports`，打开用于创建这些报告的页面
- 单击`Create Report`以生成新的诊断报告
- 报告完成时，界面中显示`Done!`
- 单击`Download report`以保存该报告
- 单击`Save File`以保存文件并完成此流程
- 完成的报告将保存到托管用于访问Web控制台的Web浏览器的系统上的`Downloads`目录

### 使用Web控制台管理系统服务
&#8195;&#8195;作为Web控制台中的特权用户，可以停止、启动、启用和重新启动系统服务。此外，还可以配置网络接口，配置防火墙服务，以及管理用户帐户等。
#### 系统电源选项
Web控制台允许重新启动或关闭系统。以特权用户身份登录Web控制台。单击导航栏上的 `Overview`访问系统电源选项，从右上角的菜单中选择需要的选项，以重启或关闭系统。
#### 控制运行中的系统服务
可以使用Web控制台中的图形工具启动、启用、禁用和停止服务：
- 单击导航栏上的`Services`，访问Web控制台中的服务初始页面
- 要管理服务，可单击位于服务初始页面顶部的`System Services`
- 在搜索栏中搜索或滚动页面以选择要管理的服务
- 单击相应的`Stop`、`Restart`或`Disallow running`(mask)以管理服务
 
#### 配置网络接口和防火墙
单击导航栏上的`Networking`管理防火墙规则和网络接口：
- 在`Interfaces`部分中，单击所需的接口名称来访问管理页面
- 管理页面的顶部显示所选设备的网络流量活动。向下滚动以查看配置设置和管理选项
- 若修改配置选项或将其添加到接口，可单击所需配置的突出显示链接
- 单击`Manual`列表选项右侧的`+`，以添加其他IP地址，在相应字段中输入IP地址和网络掩码
- 单击`Apply` 以激活新设置
- 显示画面自动切回到接口管理页面，可以在其中确认新的IP地址

#### 管理用户帐户
作为特权用户，可以在Web控制台中创建新的用户帐户：
- 单击导航栏上的`Accounts`，以查看现有的帐户
- 单击`Create New Account`以打开帐户管理页面
- 输入新帐户的信息，然后单击`Create`
- 显示画面自动切回到帐户管理页面，可以在其中确认新的用户帐户

## 从红帽客户门户网站获取帮助 
### 访问红帽客户门户网站上的支持资源
&#8195;&#8195;红帽用户门户网站[https://access.redhat.com](https://access.redhat.com)为用户提供文档、下载、工具和技术专业知识的访问途径。客户可以通过知识库搜索解决方案、常见问题和文章。可以在用户门户网站中：
- 访问官方的产品文档
- 提交和管理支持case
- 管理软件订阅和权利
- 获取软件下载、更新和评估
- 查阅可帮助您优化系统配置的工具

### 客户门户网站使用入门
&#8195;&#8195;可以通过Web浏览器访问红帽客户门户网站。本节介绍客户门户网站导览。此导览可在[https://access.redhat.com/start](https://access.redhat.com/start)找到。此导览是一个非常实用的工具，可用于探索门户网站的所有功能，以及了解如何充分利用红帽订阅。
### 使用红帽支持工具搜索知识库 
&#8195;&#8195;红帽支持工具实用程序`redhat-support-tool`提供基于文本的界面，可从系统的命令行使用此工具在客户门户网站上搜索知识库文章并提交支持案例：
- 此工具没有图形界面；由于它会与红帽客户门户网站交互，因此需要接入互联网
- `redhat-support-tool`命令可以在交互模式中使用，也可加入选项和参数作为一个命令来调用。两种方式中该工具的语法均相同。默认情况下，其程序在交互模式中启动。
- 使用`help`子命令来查看所有可用的命令
- 交互模式支持`Tab`补全，以及在父级`shell`中调用程序的功能

命令示例：
```
[user@host ~]$ redhat-support-tool
Welcome to the Red Hat Support Tool.
Command (? for help):
```
`redhat-support-tool`使用说明：
- 第一次调用时，`redhat-support-tool`会提示输入红帽客户门户网站订阅者登录信息。为避免重复提供此信息，工具会询问是否要将帐户信息存储在用户的主目录中(`~/.redhat-support-tool/redhat-support-tool.conf`)
- 如果问题都通过特定的红帽客户门户网站帐户提交，`--global`选项可以将帐户信息及其他系统范围的配置保存到`/etc/redhat-support-tool.conf`中。工具的`config`命令可修改工具配置设置
- `redhat-support-tool`命令允许订阅者搜索和显示红帽客户门户网站中的知识库内容。 知识库允许关键字搜索，与`man`命令相似。您可以输入错误代码、日志文件中的语法，或者任何关键字组合，以此生成相关解决方案文档的列表

官方初始配置和基本搜索演示：
```
[user@host ~]$ redhat-support-tool
Welcome to the Red Hat Support Tool.
Command (? for help): search How to manage system entitlements with subscription-manager
Please enter your RHN user ID: subscriber
Save the user ID in /home/student/.redhat-support-tool/redhat-support-tool.conf (y/n): y
Please enter the password for subscriber: password
Save the password for subscriber in /home/student/.redhat-support-tool/redhat-support-tool.conf (y/n): y
```
在提示用户输入必要的用户配置后，工具将继续执行原先的搜索请求：
```
Type the number of the solution to view or 'e' to return to the previous menu.
  1 [ 253273:VER] How to register and subscribe a system to the Red Hat Customer
    Portal using Red Hat Subscription-Manager
  2 [ 265523:VER] Enabling or disabling a repository using Red Hat Subscription
    Management
  3 [ 100423:VER] Why does subscription-manager list return: "No Installed
    Products found" ?
...output omitted...
Select a Solution: 1
```
&#8195;&#8195;如上所述选择文章编号`1`，系统将提示选择要阅读的文档章节。最后，使用`Q`键退出所在的章节，或重复使用它来退出`redhat-support-tool`命令：
```
Select a Solution: 1

Type the number of the section to view or 'e' to return to the previous menu.
 1 Title
 2 Issue
 3 Environment
 4 Resolution
 5 Display all sections
End of options.
Section: 1

Title
===============================================================================
How to register and subscribe a system to the Red Hat Customer Portal using Red Hat Subscription-Manager
URL:        https://access.redhat.com/solutions/253273
Created On:  None
Modified On:  2017-11-29T15:33:51Z

(END) q
Section:
Section: q

Select a Solution: q

Command (? for help): q
[user@hosts ~]# 
```
#### 根据文档ID访问知识库文章
&#8195;&#8195;使用工具的`kb`命令及知识库文档ID，直接查找在线文章。返回的文档在屏幕上滚动而不进行分页，但可以将其重定向到文件以进行保存，并使用`less`一次滚动一个屏幕。官方示例：
```
[user@host ~]$ redhat-support-tool kb 253273

Title
===============================================================================
How to register and subscribe a system to the Red Hat Customer Portal using Red Hat Subscription-Manager
URL:        https://access.redhat.com/solutions/253273
Created On:  None
Modified On:  2017-11-29T15:33:51Z

Issue
===============================================================================
*   How to register a new `Red Hat Enterprise Linux` system to the Customer Portal using `Red Hat Subscription-Manager`
...output omitted...
```
### 用红帽支持工具管理支持案例 
&#8195;&#8195;产品订阅的一个优点是能够通过红帽客户门户网站访问技术支持。根据系统的订阅支持级别，可以通过在线工具或电话联系红帽。更多信息参见 [https://access.redhat.com/site/support/policy/support_process](https://access.redhat.com/site/support/policy/support_process)。
#### 准备错误报告
在联系红帽支持部门之前，务必要为错误报告收集相关的信息： 
- 定义问题。能够清晰地陈述问题及其症状，尽可能具体，详述可重现该问题的步骤
- 收集背景信息：
    - 受影响的产品和版本是什么？准备好提供相关的诊断信息。这可能包含`sosreport`的输出
    - 对于内核问题，这可能包含系统的`kdump`崩溃转储或者崩溃系统的监控器上显示的内核回溯的数字照片
- 确定严重级别。红帽使用四个严重级别为问题分类。报告紧急和高严重级别问题后，应致电相关的当地支持中心。参见[https://access.redhat.com/site/support/contact/technicalSupport](https://access.redhat.com/site/support/contact/technicalSupport)

严重级别定义：

严重性|描述
:---|:---
紧急(严重级别1)|严重影响生产环境中使用该软件的问题。这包括生产数据丢失或生产系统故障。这种情况使业务运作暂停，也不存在其他可绕过问题的解决方法
高(严重级别2)|软件可以正常运行，但生产环境中的使用被严重削弱的问题。这种情况对业务运作有高度影响，也不存在其他可绕过问题的解决方法
中(严重级别3)|涉及生产环境或开发环境中软件部分的使用、非关键损失的问题。对于生产环境，业务会受到中低影响。业务可通过其他可绕过问题的解决方法继续正常运作。在开发环境中，这种情况将导致项目迁移至生产时出现问题
低(严重级别4)|一般使用问题，报告文档错误或未来产品增强或修改建议。在生产环境中，对业务或系统的性能或功能影响较低或没有影响。在开发环境中，对业务有中低级影响，但是业务可通过其他可绕过问题的解决方法继续正常运作

#### 通过redhat-support-tool管理错误报告
可以通过`redhat-support-tool`创建、查看、修改和关闭红帽支持案例：
- 当支持案例处于 `opened`或`maintained`状态时，用户可以附上文件或文档，如诊断报告(sosreport)。工具将上传并附加文件到案例中
- 可以通过命令选项来指定产品名称、版本、摘要、描述、严重级别和案例组等案例详细信息，也可让工具提示输入必要的信息

示例打开一个新案例，已指定了`--product`和`--version`选项：
```
[user@host ~]$ redhat-support-tool
Welcome to the Red Hat Support Tool.
Command (? for help): opencase --product="Red Hat Enterprise Linux" --version="7.0"
Please enter a summary (or 'q' to exit): System fails to run without power
Please enter a description (Ctrl-D on an empty line when complete):
When the server is unplugged, the operating system fails to continue.
 1   Urgent
 2   High
 3   Normal
 4   Low
Please select a severity (or 'q' to exit): 4
Would you like to assign a case group to this case (y/N)? N
Would see if there is a solution to this problem before opening a support case? (y/N) N
-------------------------------------------------------------------------------
Support case 01034421 has successfully been opened.
```
如果未指定`--product`和`--version`选项，将提供这些选项的选项列表：
```
[user@host ~]$ redhat-support-tool
Welcome to the Red Hat Support Tool.
Command (? for help): opencase
Do you want to use the default product - "Red Hat Enterprise Linux" (y/N)?: y
...output omitted...
29  7.4
30  7.5
31  7.6
32  8.0 Beta
Please select a version (or 'q' to exit): 32
Please enter a summary (or 'q' to exit): yum fails to install apache
Please enter a description (Ctrl-D on an empty line when complete):
yum cannot find correct repo
 1   Urgent
 2   High
 3   Normal
 4   Low
Please select a severity (or 'q' to exit): 4
Would you like to use the default (Ungrouped Case) Case Group (y/N)? : y
Would you like to see if there's a solution to this problem before opening a support case? (y/N) N
-------------------------------------------------------------------------------
Support case 010355678 has successfully been opened. 
```
#### 将诊断信息附加到支持案例
包含诊断信息可以更快地解决问题，建议打开案例时附上`sosreport`：
- `sosreport`命令生成压缩的`tar`存档，内含从运行中的系统收集的诊断信息
- 如果之前创建过存档，则`redhat-support-tool`将提示包含该存档

示例如下：
```
Please attach a SoS report to support case 01034421. Create a SoS report as
the root user and execute the following command to attach the SoS report
directly to the case:
 redhat-support-tool addattachment -c 01034421 path to sosreport

Would you like to attach a file to 01034421 at this time? (y/N) N
Command (? for help):
```
&#8195;&#8195;如果不存在当前的`SoS`报告，管理员可以稍后生成并附加一份报告。使用`redhat-support-tool addattachment`命令来附加报告。订阅者可以查看、修改和关闭支持案例：
```
Command (? for help): listcases

Type the number of the case to view or 'e' to return to the previous menu.
 1 [Waiting on Red Hat]  System fails to run without power
No more cases to display
Select a Case: 1

Type the number of the section to view or 'e' to return to the previous menu.
 1 Case Details
 2 Modify Case
 3 Description
 4 Recommendations
 5 Get Attachment
 6 Add Attachment
 7 Add Comment
End of options.
Option: q

Select a Case: q

Command (? for help):q

[user@host ~]$ redhat-support-tool modifycase --status=Closed 01034421
Successfully updated case 01034421
[user@host ~]$
```
红帽支持工具更多说明：
- 红帽支持工具具有高级应用诊断和分析功能：
    - 利用内核崩溃转储核心文件，`redhat-support-tool`可以创建和提取回溯追踪。回溯追踪是崩溃转储点处活动堆栈帧的报告，也能提供现场诊断
    - `redhat-support-tool`的选项之一是打开支持案例
- 该工具也提供日志文件分析功能：
    - 通过工具的`analyze`命令，可以解析许多类型的日志文件（如操作系统、JBoss、Python、Tomcat 和 oVirt）来识别问题症状
    - 日志文件可以单独进行查看和诊断
    - 与崩溃转储或日志文件等原始数据相比，提供预处理过的分析可以更加快速地创建支持案例并提交给工程师

### 加入红帽开发者计划
红帽提供的另一个有用资源是红帽开发者计划：
- 此计划托管于[https://developer.redhat.com](https://developer.redhat.com)，提供开发专用红帽软件订阅权利、相关文档，以及我们微服务、无服务器计算、Kubernetes和Linux领域的专家推出的优质图书
- 另外，也提供博客、即将举办的活动与培训的信息链接和其他帮助资源，以及红帽客户门户网站的链接 
- 注册免费，注册地址：[https://developer.redhat.com/register](https://developer.redhat.com/register)

## 通过红帽智能分析工具检测和解决问题 
### 红帽智能分析工具简介
&#8195;&#8195;红帽智能分析工具是一种预测分析工具，可帮助用户识别和修复基础架构中运行红帽产品的系统上面临的安全性、性能、可用性和稳定性威胁：
- 智能分析工具作为软件即服务(SaaS)产品提供，因此可以快速部署和扩展它，没有额外的基础架构要求
- 可以立即利用特定于已部署系统的红帽最新建议和更新 
- 红帽会定期更新智能分析工具使用的知识库，这些知识库基于常见的支持风险、安全漏洞、已知错误的配置，以及红帽识别的其他问题
- 缓解或修复这些问题的措施会得到红帽的检验和验证
- 有了这种支持，用户可以在问题成为更大问题之前主动识别问题，确定其优先级并加以解决
- 对于检测到的每个问题，智能分析工具提供所呈现风险的估测以及有关如何缓解或修复问题的建议：
    - 这些建议可提供诸如`Ansible Playbook`或易于阅读的分步骤说明等资料来帮助用户解决问题
- 智能分析工具会针对注册到服务的每个系统定制推荐内容：
    - 安装各个客户端系统时会一同安装一个代理，它将收集有关系统运行时配置的元数据
    - 使用`sosreport`命令向红帽支持部门提供数据时应包含这一数据，以便能解决支持票据
    - 可以限制或模糊处理客户端发送的数据。这会使某些分析规则无法运作，具体取决于用户的限制
- 在用户注册服务器并且服务器完成初始系统元数据同步之后不久，就能够在红帽云门户网站上的智能分析工具控制台中看到用户的服务器和对应的建议。
- 智能分析工具目前为下列红帽产品提供预测分析和建议：
    - 红帽企业Linux 6.4及更高版本
    - 红帽企业虚拟化4及更高版本
    - 红帽OpenShift容器平台
    - 红帽OpenStack平台7及更高版本 
- 使用智能分析工具注册系统时，它会立即将有关其当前配置的元数据发送到智能分析工具平台
- 注册后，系统会定期更新提供给智能分析工具的元数据。系统会使用TLS加密发送元数据，以便在传输中保护元数据
- 智能分析工具收到数据后，会对其进行分析，并在位于[https://cloud.redhat.com/insights](https://cloud.redhat.com/insights)的智能分析工具Web控制台上显示结果

### 安装智能分析工具客户端
&#8195;&#8195;智能分析工具已作为订阅的一部分随附于RHEL8中。旧版的RHEL服务器需要在系统上安装 `insights-client`软件包。如果系统通过客户门户网站订阅管理服务注册了软件权利，则可以使用一个命令来激活智能分析工具。使用`insights-client --register`命令来注册系统：
```
[root@host ~]# insights-client --register
```
&#8195;&#8195;智能分析工具客户端定期更新提供给智能分析工具的元数据。用户随时可以使用 `insights-client`命令刷新客户端的元数据：
```
[root@host ~]# insights-client
Starting to collect Insights data for host.example.com
Uploading Insights data.
Successfully uploaded report for host.example.com.
View details about this system on cloud.redhat.com:
https://cloud.redhat.com/insights/inventory/dc480efd-4782-417e-a496-cb33e23642f0
```
#### 将RHEL系统注册到智能分析工具
使用红帽订阅管理服务，以交互方式注册系统：
```
[root@host ~]# subscription-manager register --auto-attach
```
&#8195;&#8195;确保系统上已安装`insights-client`软件包(在RHEL8系统上不需要此步骤)。在RHEL 7中，此软件包位于`rhel-7-server-rpms`中：
```
[root@host ~]# yum install insights-client
```
&#8195;&#8195;使用`insights-client --register`命令，将系统注册到智能分析工具服务并上传初始系统元数据:
```
[root@host ~]# insights-client --register
```
&#8195;&#8195;确认系统在位于[https://cloud.redhat.com/insights](https://cloud.redhat.com/insights)的智能分析工具Web控制台中的`Inventory`下可见。
### 智能分析工具控制台导航
&#8195;&#8195;智能分析工具提供一系列服务供您通过位于[https://cloud.redhat.com/insights](https://cloud.redhat.com/insights)的Web控制台访问。
#### 使用顾问服务检测配置问题
可以使用顾问服务检测配置问题，顾问服务将会报告对用户系统有影响的配置问题。可以从`Advisor`→ `Recommentations`菜单访问该服务：
- 对于每个问题，智能分析工具都会提供附加信息，以帮助了解问题，确定解决问题的工作的优先级，确定可用的缓解或修复措施，并通过`Ansible Playbook`自动解决问题
- 智能分析工具还会提供客户门户上的知识库文章链接
- 顾问服务会分两类评估某个问题给用户系统带来的风险：
    - 总体风险：表示问题对用户的系统的影响
    - 变动风险：表示修复措施对用户的系统的影响。例如可能需要重启系统 

#### 使用漏洞服务进行安全性评估
&#8195;&#8195;漏洞服务会报告对用户的系统有影响的常见的漏洞和风险(CVE)。可以从`Vulnerability`→`CVEs`菜单访问该服务：
- 对于每个CVE，智能分析工具均会提供附加信息，并列出暴露在风险中的系统
- 可以单击`Remediate`来创建用于修复的`Ansible Playbook`

#### 使用合规性服务分析合规性
&#8195;&#8195;合规性服务会分析用户的系统，并根据`OpenSCAP`策略报告系统的合规性水平。`OpenSCAP`项目会实施一些工具，以便根据一套规则检查系统的合规性。智能分析工具会提供一些规则来对照不同策略（如支付卡行业数据安全标准(PCI DSS)评估用户的系统。
#### 使用补丁服务更新软件包
&#8195;&#8195;补丁服务会列出适用于用户的系统的红帽产品公告。该服务还会生成供用户运行的`Ansible Playbook`，用来更新与适用公告相关联的RPM软件包：
- 访问特定系统的公告列表，请使用`Patch`→`Systems`菜单
- 单击系统的`Apply all applicable advisories`，以生成`Ansible Playbook`

#### 使用偏移服务对系统进行比较
&#8195;&#8195;利用偏移服务，可以比较系统或系统历史记录。此项服务可以将系统与类似系统或之前的系统状态进行比较，从而帮用户对该系统进行故障排除。可以从`Drift`→`Comparison`菜单访问该服务。
#### 使用策略服务触发警报
&#8195;&#8195;利用策略服务，可以创建用于监控系统的规则，并在系统不符合规则时发送警报。每次系统同步其元数据时，智能分析工具都会对规则进行评估。可以从`Policies`菜单访问该服务。
#### 访问清单和修复Playbook并监控订阅
`Inventory`页面中提供用户已注册到红帽智能分析工具的系统列表：
- `Last seen`列显示各个系统最近一次更新元数据的时间。单击系统名称即可查看其详细信息并直接访问相应系统的顾问服务、漏洞服务、合规性服务和补丁服务
- `Remediations`页面会列出用户创建的用于修复的所有`Ansible Playbook`。用户可以从该页面下载`playbook`
- 使用`Subscription Watch`页面，可以监控用户的红帽订阅使用情况

## 练习