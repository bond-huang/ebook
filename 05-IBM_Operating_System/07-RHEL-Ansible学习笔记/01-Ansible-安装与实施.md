# Ansible-安装与实施
官方参考链接：
- [How to download and install Red Hat Ansible Engine](https://access.redhat.com/articles/3174981)
- [Red Hat Ansible Automation Platform安装指南](https://access.redhat.com/documentation/zh-cn/red_hat_ansible_automation_platform/2.2/html/red_hat_ansible_automation_platform_installation_guide/index)

## 下载与安装
### 控制节点
#### RHEL9下载安装
下载最新的RHEL9.0版本，官方下载链接：[Download Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/download)

下载完成后配置本地YUM源：
```
[root@localhost ~]# yum repolist
Updating Subscription Management repositories.

repo id                                 repo name
redhat9_app                             redhat9_app
redhat9_os                              redhat9_os
```
#### 安装Python
&#8195;&#8195;如果是RHEL8，Ansible可以自动使用`platform-python`软件包，该软件包支持使用Python的系统实用程序。此次使用的是RHEL9，默认是没有了，光盘里面也没有。但是有Python39，暂时不安装。
#### 注册系统
将系统注册到红帽订阅管理工具：
```
[root@localhost ~]# subscription-manager register
Registering to: subscription.rhsm.redhat.com:443/subscription
Username: 
Password:
The system has been registered with ID: 
The registered system name is: localhost
```
#### 订阅仓库管理
查看系统通过订阅获取的仓库：
```
[root@redhat9 ~]# subscription-manager repos --list
This system has no repositories available through subscriptions.
```
查看subscription-manager版本：
```
[root@redhat9 ~]# subscription-manager version
server type: Red Hat Subscription Management
subscription management server: 4.0.18-3
subscription management rules: 5.41
subscription-manager: 1.29.26-3.el9_0
```
查看是否有Ansible相关订阅：
```
[root@redhat9 ~]# subscription-manager list --available |grep -i ansible
                     Red Hat Ansible Engine
                     Red Hat Ansible Automation Platform
```
记住Pool ID将订阅添加到系统：
```
[root@redhat9 ~]# subscription-manager attach --pool=2c9xxxxf82fexxxxx183044cccxxxxxx
Successfully attached a subscription for: Red Hat Developer Subscription for Individuals
```
查看当前仓库：
```
[root@redhat9 ~]# yum repolist
Updating Subscription Management repositories.
repo id                                            repo name
redhat9_app                                        redhat9_app
redhat9_os                                         redhat9_os
rhel-9-for-x86_64-appstream-rpms                   Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
rhel-9-for-x86_64-baseos-rpms                      Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)
```
#### 启用Ansible引擎库
启动Ansible相关仓库：
```
[root@redhat9 ~]# subscription-manager repos --enable \
> ansible-automation-platform-2.2-for-rhel-9-x86_64-rpms
Repository 'ansible-automation-platform-2.2-for-rhel-9-x86_64-rpms' is enabled for this system.
```
然后安装Ansible即可：
```
[root@redhat9 ~]# yum install ansible
```
安装的RHEL9系统版本里面已经有Ansible了：
```
[root@redhat9 ~]# sudo ansible --version
ansible [core 2.12.2]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share                                              /ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/co                                              llections
  executable location = /bin/ansible
  python version = 3.9.10 (main, Feb  9 2022, 00:00:00) [GCC 11.2.1 20220127 (Re                                              d Hat 11.2.1-9)]
  jinja version = 2.11.3
  libyaml = True
```
调用本地主机上的`setup`模块，检索`ansible_python_version`的值：
```
[root@redhat9 ~]# ansible -m setup localhost | grep ansible_python_version
        "ansible_python_version": "3.9.10",
```
### 受控节点
&#8195;&#8195;受管主机不需要安装特殊的代理。Ansible控制节点使用标准的网络协议连接受管主机，从而确保系统处于指定的状态。受管主机可能要满足一些要求：
- Linux和UNIX受管主机需要安装有`Python2`（版本2.6或以上）或`Python3`（版本3.5或以上），这样才能运行大部分的模块
- 对于RHEL8，可以依靠`platform-python`软件包。也可以启用并安装python36应用流（或 python27应用流）

我采用RHEL8的受控节点，platform-python信息如下：
```
[root@redhat8 ~]# yum list installed platform-python
Updating Subscription Management repositories.
Installed Packages
platform-python.x86_64                                          3.6.8-1.el8                                          @anaconda
```
## 实施Ansible Playbook
### 定义清单
&#8195;&#8195;清单定义Ansible将要管理的一批主机，这些主机也可以分配到组中，组可以包含子组，主机也可以是多个组的成员。清单还可以设置应用到它所定义的主机和组的变量。可以通过两种方式定义主机清单：
- 静态主机清单可以通过文本文件来定义
- 动态主机清单可以根据需要使用外部信息提供程序通过脚本或其他程序来生成

#### 静态清单
##### 静态清单指定受管主机
&#8195;&#8195;可以使用多种不同的格式编写静态清单，包括`INI`样式或`YAML`。`INI`样式最简单的静态清单文件是受管主机的主机名或IP地址的列表，每行一个：
```
192.168.100.131
web.big1000.com
db2.big1000.com
```
示例主机清单定义了多个主机组，并且主机可以在多个组中：
```ini
[webservers]
web.big1000.com
ebook.big1000.com

[db-servers]
db2.big1000.com
oracle.big1000.com
192.168.100.130

[prod-servers]
ebook.big1000.com
db2.big1000.com

[development]
192.168.100.131
```
说明：
- `ansible all --list-hosts`中`all`主机组含有清单中明确列出的每一个主机
- `ansible ungrouped --list-hosts`中`ungrouped`主机组含有清单中明确列出、但不属于任何其他组的每一个主机

##### 定义嵌套组
&#8195;&#8195;Ansible主机清单可以包含由多个主机组构成的组。这通过创建后缀为`:children`的主机组名称来实现。示例创建名为`prod-servers`的新组，它包含来自`webservers`和`db-servers`组的所有主机：
```ini
[webservers]
web.big1000.com
ebook.big1000.com

[db-servers]
db2.big1000.com
oracle.big1000.com
192.168.100.130

[prod-servers:children]
webservers
db-servers
```
&#8195;&#8195;一个组可以同时包含受管主机和子组作为其成员。例如，在上面的清单中，可以添加一个拥有自己的受管主机列表的`[prod-servers]`部分。这一列表中的主机将与`north-america`组从其子组中继承的其他主机合并。
##### 简化主机规格
&#8195;&#8195;如果主机过多，并且有明显规律可循，可以通过指定主机名称或`IP`地址的范围来简化Ansible主机清单。可以指定数字或字母范围。语法如下所示：
```
[START:END]
```
范围匹配从`START`到`END`(包含)的所有值。示例：
- `192.168.[4:7].[0:255]`匹配`192.168.4.0/22`网络中的所有IPv4地址(192.168.4.0到192.168.7.255)
- `web[01:10].big1000.com`匹配名为`web01.big1000.com`到`web10.big1000.com`的所有主机(如果数字范围中包含前置零，其模式中会使用它们，所以不会匹配web1)
- `[a:c].big1000.com`匹配名为`a.big100.com`、`b.big100.com`和`c.big100.com`的主机
- `2001:db8::[a:f]`匹配从`2001:db8::a`到`2001:db8::f`的所有IPv6地址

##### 验证清单
验证主机redhat8是否在清单中：
```
[root@redhat9 ~]# ansible redhat8 --list-hosts
  hosts (1):
    redhat8
```
列出组testhosts中的所有主机：
```
[root@redhat9 ~]# ansible testhosts --list-hosts
  hosts (2):
    redhat8
    192.168.100.130
```
&#8195;&#8195;如果清单中含有名称相同的主机和主机组，`ansible`命令将显示警告并以主机作为其目标。主机组则被忽略。应确保主机组不使用与清单中主机相同的名称。
##### 覆盖清单的位置
&#8195;&#8195;`/etc/ansible/hosts`文件是系统的默认静态清单文件。通常不使用该文件，而是在Ansible配置文件中为清单文件定义一个不同的位置。
##### 在清单中定义变量
&#8195;&#8195;可以在主机清单文件中指定playbook使用的变量值。这些变量仅应用到特定的主机或主机组。通常，最好在特殊目录中定义这些库存变量，而不要直接在清单文件中定义。
#### 动态清单
&#8195;&#8195;可以使用外部数据库提供的信息动态生成Ansible清单信息。开源社区编写了许多可从上游 Ansible项目获取的动态清单脚本。
### 管理Ansible配置文件
#### 配置Ansible
Ansible从控制节点上多个可能的位置之一选择其配置文件：
- `/etc/ansible/ansible.cfg`：ansible软件包的此位置会有一个基础配置文件。如果找不到其他配置文件，则使用此文件
- `~/.ansible.cfg`：Ansible在用户的主目录中查找`.ansible.cfg`文件。如果存在此配置并且当前工作目录中也没有`ansible.cfg`文件，则使用此配置取代`/etc/ansible/ansible.cfg`
- `./ansible.cfg`：如果执行`ansible`命令的目录中存在`ansible.cfg`文件，则使用它，而不使用全局文件或用户的个人文件

&#8195;&#8195;推荐的做法是在要运行`Ansible`命令的目录中创建`ansible.cfg`文件。此目录中也将包含任何供用户的`Ansible`项目使用的文件，如清单和`playbook`。这是用于Ansible 配置文件的最常用位置。在实践中，很少会使用`~/.ansible.cfg`或`/etc/ansible/ansible.cfg`文件。     
&#8195;&#8195;可以通过将不同的配置文件放在不同的目录中，然后从适当目录执行Ansible命令，以此利用配置文件；随着配置文件数量的增加，这种方法难以管理。可以通过`ANSIBLE_CONFIG`环境变量定义配置文件的位置，Ansible将使用变量所指定的配置文件，而不用上面提及的任何配置文件。
#### 配置文件优先级
&#8195;&#8195;配置文件的搜索顺序与上述列表相反。位于搜索顺序中的第一个文件是Ansible选择的文件。Ansible仅使用它找到的第一个文件中的配置设置。
- `ANSIBLE_CONFIG`环境变量指定的任何文件都会覆盖所有其他配置文件
- 如果没有设置该变量，则接下来检查运行ansible命令的目录中是否有`ansible.cfg`文件
- 如果不存在上述文件，则检查用户的主目录中是否有`.ansible.cfg`文件
- 只有在找不到其他配置文件时，才使用全局`/etc/ansible/ansible.cfg`文件
- 如果`/etc/ansible/ansible.cfg`配置文件不存在，Ansible包含它使用的默认值

&#8195;&#8195;可以运行`ansible --version`命令来清楚地确认所安装的Ansible版本，以及正在使用的配置文件。命令示例如下：
```
[root@redhat9 ~]# ansible --version
ansible [core 2.12.2]
  config file = /etc/ansible/ansible.cfg
...output omitted...
```
执行`Ansible`命令时使用`-v`选项是显示活动的Ansible配置文件的另一种方式：
```
[root@redhat9 ~]# ansible servers --list-hosts -v
Using /etc/ansible/ansible.cfg as config file
...output omitted...
```
#### 管理配置文件中的设置
&#8195;&#8195;Ansible配置文件由几个部分组成，每一部分含有以键值对形式定义的设置。部分的标题以方括号括起。对于基本操作，使用以下两部分：
- `[defaults]`部分设置Ansible操作的默认值
- `[privilege_escalation]`配置Ansible如何在受管主机上执行特权升级

例如，下面是典型的`ansible.cfg`文件：
```ini
[defaults]
inventory = ./inventory
remote_user = user
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```
此文件中的指令如下表：

指令|描述
:---:|:---
inventory|指定清单文件的路径
remote_user|要在受管主机上登录的用户的名称。如果未指定，则使用当前用户的名称
ask_pass|是否提示输入SSH密码。如果使用SSH公钥身份验证，则可以是false
become|连接后是否自动在受管主机上切换用户(通常切换为root)。也可以通过play来指定
become_method|如何切换用户(通常为sudo，sudo默认设置，也可选择su)
become_user|要在受管主机上切换到的用户(通常是root，root是默认值)
become_ask_pass|是否需要为become_method提示输入密码。默认为false

#### 配置连接
&#8195;&#8195;Ansible需要知道如何与其受管主机通信。更改配置文件的一个最常见原因是为了控制Ansible使用什么方法和用户来管理受管主机。需要的一些信息包括：
- 列出受管主机和主机组的清单的位置
- 要使用哪一种连接协议来与受管主机通信(默认为SSH)，以及是否需要非标准网络端口来连接服务器
- 要在受管主机上使用哪一远程用户；可以是root用户或者某一非特权用户
- 如果远程用户为非特权用户，Ansible需要知道它是否应尝试将特权升级为root以及如何进行升级(如使用sudo)
- 是否提示输入SSH密码或sudo密码以进行登录或获取特权

##### 清单位置
&#8195;&#8195;在`[defaults]`部分中，`inventory`指令可以直接指向某一静态清单文件，或者指向含有多个静态清单文件和动态清单脚本的某一目录。
##### 连接设置
连接设置：
- 默认情况下，Ansible使用SSH协议连接受管主机。控制Ansible如何连接受管主机的最重要参数在`[defaults]`部分中设置。
- 默认情况下，Ansible尝试连接受管主机时使用的用户名与运行Ansible命令的本地用户相同。若要指定不同的远程用户，将`remote_user`参数设置为该用户名
- 如果为运行Ansible的本地用户配置了SSH私钥，使得它们能够在受管主机上进行远程用户的身份验证，则 Ansible将自动登录。如果不是这种情况，可以通过设置指令`ask_pass = true`，将Ansible配置为提示本地用户输入由远程用户使用的密码
- 如果在使用一个Linux控制节点，并对受管主机使用OpenSSH，如果可以使用密码以远程用户身份登录，那么或许可以设置基于SSH密钥的身份验证，从而能够设置`ask_pass = false`
- 第一步是确保在`~/.ssh`中为控制节点上的用户配置了SSH密钥对。可以运行`ssh-keygen`命令来实现这个目标

&#8195;&#8195;如果是单一现有受管主机，可以在受管主机上安装公钥，并使用`ssh-copy-id`命令在本地的`~/.ssh/known_hosts`文件中填充其主机密钥，如下所示：
```
[root@redhat8 ~]# ssh-keygen -t dsa
Generating public/private dsa key pair.
Enter file in which to save the key (/root/.ssh/id_dsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_dsa.
Your public key has been saved in /root/.ssh/id_dsa.pub.
The key fingerprint is:
...output omitted...
[root@redhat8 ~]# ssh-copy-id root@redhat9
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_dsa.pub"
The authenticity of host 'redhat9 (192.168.100.133)' can't be established.
ECDSA key fingerprint is SHA256:DCK9bY2lCTvdQooqufMRIHemI2vxLNT2knhj1fKIyY0.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@redhat9's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@redhat9'"
and check to make sure that only the key(s) you wanted were added.
```
##### 升级特权
&#8195;&#8195;鉴于安全性和审计原因，Ansible可能需要先以非特权用户身份连接远程主机，然后再通过特权升级获得root用户身份的管理权限。这可以在Ansible配置文件的`[privilege_escalation]`部分中设置。
- 要默认启用特权升级，可在配置文件中设置指令`become = true`。即使默认为该设置，也可以在运行临时命令或Ansible Playbook时通过各种方式覆盖它
- `become_method`指令指定如何升级特权。有多个选项可用，默认为`sudo`
- `become_user`指令指定要升级到的用户，默认为root
- 如果所选的`become_method`机制要求用户输入密码才能升级特权，可以在配置文件中设置`become_ask_pass = true`指令

&#8195;&#8195;在RHEL7上，`/etc/sudoers`的默认配置允许`wheel`组中的所有用户在输入密码后使用`sudo`成为`root`用户。若要让用户（示例中为ansible）在不输入密码前提下使用`sudo`成为`root`，一种办法是将含有适当指令的文件安装到`/etc/sudoers.d`目录（归root 所有，八进制权限为0400）：
```
## password-less sudo for Ansible user
ansible ALL=(ALL) NOPASSWD:ALL
```
&#8195;&#8195;示例`ansible.cfg`文件假定用户可以通过基于SSH密钥的身份验证以`ansible`用户身份连接受管主机，并且`ansible`可以使用`sudo`以`root`用户身份运行命令而不必输入密码：
```
[defaults]
inventory = ./inventory
remote_user = ansible
ask_pass = false

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```
#### 非SSH连接
&#8195;&#8195;默认情况下，Ansible用于连接受管主机的协议设置为`smart`，它会确定使用SSH的最高效方式。可以通过多种方式将其设置为其他的值。     
&#8195;&#8195;例如，默认使用SSH的规则有一个例外。如果用户目录中没有`localhost`，Ansible将设置一个隐式`localhost`条目以允许运行以`localhost`为目标的临时命令和`playbook`。这一特殊清单条目不包括在`all`或`ungrouped`主机组中。此外，Ansible不使用smart SSH连接类型，而是利用默认的特殊local连接类型来进行连接。
```
[root@redhat9 ~]# ansible localhost --list-hosts
  hosts (1):
    localhost
```
&#8195;&#8195;`local`连接类型忽略`remote_user`设置，并且直接在本地系统上运行命令。如果使用了特权升级，它会在运行 `sudo`时使用运行Ansible命令的用户帐户，而不是`remote_user`。如果这两个用户具有不同的`sudo`特权，这可能会导致混淆。
- 如果要确保像其他受管主机一样使用SSH连接`localhost`，一种方法是在清单中列出它。但是，这会将它包含在 `all`和`ungrouped`组中，可能不希望如此。
- 另一种办法是更改用于连接`localhost`的协议。执行此操作的最好方法是为`localhost`设置 `ansible_connection`主机变量。为此，要在运行Ansible命令的目录中创建`host_vars`子目录。在该子目录中，创建名为`localhost`的文件，其应含有`ansible_connection: smart`这一行。这将确保对`localhost`使用`smart (SSH)`连接协议，而非`local`。
- 也可以通过另一种变通办法来使用它。如果清单中列有`127.0.0.1`，则默认情况下，将使用smart来连接它。也可以创建一个含有`ansible_connection: local`这一行的`host_vars/127.0.0.1`文件，它会改为使用`local`。
#### 配置文件注释
Ansible配置文件允许使用两种注释字符：井号或编号符号(#)以及分号(;)。
- 位于行开头的编号符号会注释掉整行。它不能和指令位于同一行中
- 分号字符可以注释掉所在行中其右侧的所有内容。它可以和指令位于同一行中，只要该指令在其左侧

#### 示例练习
创建目录：
```
[root@redhat9 ~]# mkdir ~/deploy-manage
[root@redhat9 ~]# cd ~/deploy-manage
[root@redhat9 deploy-manage]#
```
编辑`ansible.cfg`文件，内容如下：
```ini
[defaults]
inventory = ./inventory

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = true
```
编辑`inventory`文件，内容如下：
```
[myself]
localhost

[redhata]
redhat8a

[redhatb]
redhat8b

[redhat:children]
redhata
redhatb
```
运行`ansible --list-hosts`命令，以验证是否提示输入`sudo`密码：
```
[root@redhat9 deploy-manage]# ansible redhatb --list-hosts
BECOME password:
  hosts (1):
    redhat8b
```
### 运行临时命令
#### 使用Ansible运行临时命令