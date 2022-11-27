# Ansible-安装Ansible
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
repo id                              repo name
redhat9_app                          redhat9_app
redhat9_os                           redhat9_os
rhel-9-for-x86_64-appstream-rpms     Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
rhel-9-for-x86_64-baseos-rpms        Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)
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
## 添加互信
### 添加用户
控制节点和受控节点上都添加ansible用户：
```
[root@redhat8 ~]# useradd ansible
[root@redhat8 ~]# cat /etc/passwd |grep ansible
ansible:x:1011:1011::/home/ansible:/bin/bash
```
可以设置密码：
```
[root@redhat8 ~]# passwd ansible
Changing password for user ansible.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```
用户设置为可以ssh访问。
### 添加host表
在控制节点和受控节点上都添加对方信息，示例：
```
[ansible@redhat9 ~]$ cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.100.133 redhat9
192.168.100.130 redhat8
```
### 添加互信
&#8195;&#8195;如果是单一现有受管主机，可以在受管主机上安装公钥，并使用`ssh-copy-id`命令在本地的`~/.ssh/known_hosts`文件中填充其主机密钥，如下所示：
```
[ansible@redhat8 ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ansible/.ssh/id_rsa):
Created directory '/home/ansible/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ansible/.ssh/id_rsa.
Your public key has been saved in /home/ansible/.ssh/id_rsa.pub.
The key fingerprint is:
...output omitted...
```
将密钥拷贝到对端系统：
```
[ansible@redhat8 ~]$ ssh-copy-id ansible@redhat9
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ansible/.ssh/id_rsa.pub"
The authenticity of host 'redhat9 (192.168.100.133)' can't be established.
ECDSA key fingerprint is SHA256:DCK9bY2lCTvdQooqufMRIHemI2vxLNT2knhj1fKIyY0.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
ansible@redhat9's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'ansible@redhat9'"
and check to make sure that only the key(s) you wanted were added.
```
验证是否可以：
```
[ansible@redhat8 ~]$ ssh ansible@redhat9
Register this system with Red Hat Insights: insights-client --register
Create an account or view all your systems at https://red.ht/insights-dashboard
Last login: Mon Sep  5 15:16:08 2022 from 192.168.100.1
[ansible@redhat9 ~]$ 
```
在另外一个节点也执行一遍，就可以互相免密访问了。查看`known_hosts`示例：
```
[ansible@redhat9 ~]$ cat .ssh/known_hosts
redhat8 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AbsjHS25kvfAWcZ3KmZVXQ/WXCT4V5m
192.168.100.131 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5PIXbsjHS25kvfAWcZ3KmZjVXQ/WXCT4
```
## 检查Ansible配置选项
### 查看配置选项
&#8195;&#8195;如果想要查找配置文件中的可用选项，使用`ansible-config list`命令。它将显示可用配置选项及其默认设置的详尽列表。根据安装的Ansible版本以及控制节点上是否有任何其他Ansible插件，此列表可能会有所不同。    
&#8195;&#8195;`ansible-config list`显示的每个选项会有多个与之相关联的密匙对。这些键值对提供有关该选项的工作方式的信息。例如，选项`ACTION_WARNINGS`显示以下键值对：

键|值|用途
:---:|:---|:---
description|默认情况下，从任务操作(模块或操作插件)收到时，Ansible将发出警告。通过将此设置调整为False可以使这些警告静音|描述此配置选项的用途
type|boolean|该选项的类型是什么：boolean是指true-false值
default|true|此选项的默认值
version_added|2.5|为实现向后兼容性而添加了此选项的Ansible版本
ini|`{ key: action_warnings, section: defaults }`|类似INI清单文件的哪部分包含此选项，以及该选项在配置文件中的名称(defaults 部分中的 action_warnings)
env|ANSIBLE_ACTION_WARNINGS|如果设置了此环境变量，它将覆盖配置文件中所做的任何选项设置

### 确定修改的配置选项
&#8195;&#8195;使用配置文件时，可能想找出哪些选项已设置为与内置默认值不同的值。可以运行`ansible-config dump -v --only-changed`命令来完成此操作：
- `-v`选项显示处理该命令时使用`ansible.cfg`文件的位置
- `ansible-config`命令遵循前面提到的`ansible`命令的相同优先顺序
- 根据`ansible.cfg`文件的位置和从中运行`ansible-config`命令的目录，输出将有所不同

&#8195;&#8195;下面示例中，有单独的一个ansible配置文件位于`/etc/ansible/ansible.cfg`。`ansible-config`命令首先从学员的主目录运行，然后从工作目录运行，结果相同：
```
[user@controlnode ~]$ ansible-config dump -v --only-changed
Using /etc/ansible/ansible.cfg as config file
DEFAULT_ROLES_PATH(/etc/ansible/ansible.cfg) = [u'/etc/ansible/roles', u'/usr/share/ansible/roles']

[user@controlnode ~]$ cd /home/student/workingdirectory
[user@controlnode workingdirectory]$ ansible-config dump -v --only-changed
Using /etc/ansible/ansible.cfg as config file
DEFAULT_ROLES_PATH(/etc/ansible/ansible.cfg) = [u'/etc/ansible/roles', u'/usr/share/ansible/roles']
```
&#8195;&#8195;但是，如果工作目录中有自定义的`ansible.cfg`文件，则上述命令将根据其运行位置和相对`ansible.cfg`文件显示信息：
```
[user@controlnode ~]$ ansible-config dump -v --only-changed
Using /etc/ansible/ansible.cfg as config file
DEFAULT_ROLES_PATH(/etc/ansible/ansible.cfg) = [u'/etc/ansible/roles', u'/usr/share/ansible/roles']

[user@controlnode ~]$ cd /home/student/workingdirectory
[user@controlnode workingdirectory]$ cat ansible.cfg
[defaults]
inventory = ./inventory
remote_user = devops

[user@controlnode workingdirectory]$ ansible-config dump -v --only-changed
Using /home/student/workingdirectory/ansible.cfg as config file
DEFAULT_HOST_LIST(/home/student/workingdirectory/ansible.cfg) = [u'/home/student/workingdirectory/inventory']
DEFAULT_REMOTE_USER(/home/student/workingdirectory/ansible.cfg) = devops
```
## 待补充