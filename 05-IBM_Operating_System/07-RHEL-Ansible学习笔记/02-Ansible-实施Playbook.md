# 实施Ansible Playbook
## 定义清单
&#8195;&#8195;清单定义Ansible将要管理的一批主机，这些主机也可以分配到组中，组可以包含子组，主机也可以是多个组的成员。清单还可以设置应用到它所定义的主机和组的变量。可以通过两种方式定义主机清单：
- 静态主机清单可以通过文本文件来定义
- 动态主机清单可以根据需要使用外部信息提供程序通过脚本或其他程序来生成

### 静态清单
#### 静态清单指定受管主机
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

#### 定义嵌套组
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
#### 简化主机规格
&#8195;&#8195;如果主机过多，并且有明显规律可循，可以通过指定主机名称或`IP`地址的范围来简化Ansible主机清单。可以指定数字或字母范围。语法如下所示：
```
[START:END]
```
范围匹配从`START`到`END`(包含)的所有值。示例：
- `192.168.[4:7].[0:255]`匹配`192.168.4.0/22`网络中的所有IPv4地址(192.168.4.0到192.168.7.255)
- `web[01:10].big1000.com`匹配名为`web01.big1000.com`到`web10.big1000.com`的所有主机(如果数字范围中包含前置零，其模式中会使用它们，所以不会匹配web1)
- `[a:c].big1000.com`匹配名为`a.big100.com`、`b.big100.com`和`c.big100.com`的主机
- `2001:db8::[a:f]`匹配从`2001:db8::a`到`2001:db8::f`的所有IPv6地址

#### 验证清单
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
#### 覆盖清单的位置
&#8195;&#8195;`/etc/ansible/hosts`文件是系统的默认静态清单文件。通常不使用该文件，而是在Ansible配置文件中为清单文件定义一个不同的位置。
#### 在清单中定义变量
&#8195;&#8195;可以在主机清单文件中指定playbook使用的变量值。这些变量仅应用到特定的主机或主机组。通常，最好在特殊目录中定义这些库存变量，而不要直接在清单文件中定义。
### 动态清单
&#8195;&#8195;可以使用外部数据库提供的信息动态生成Ansible清单信息。开源社区编写了许多可从上游 Ansible项目获取的动态清单脚本。
## 管理Ansible配置文件
### 配置Ansible
Ansible从控制节点上多个可能的位置之一选择其配置文件：
- `/etc/ansible/ansible.cfg`：ansible软件包的此位置会有一个基础配置文件。如果找不到其他配置文件，则使用此文件
- `~/.ansible.cfg`：Ansible在用户的主目录中查找`.ansible.cfg`文件。如果存在此配置并且当前工作目录中也没有`ansible.cfg`文件，则使用此配置取代`/etc/ansible/ansible.cfg`
- `./ansible.cfg`：如果执行`ansible`命令的目录中存在`ansible.cfg`文件，则使用它，而不使用全局文件或用户的个人文件

&#8195;&#8195;推荐的做法是在要运行`Ansible`命令的目录中创建`ansible.cfg`文件。此目录中也将包含任何供用户的`Ansible`项目使用的文件，如清单和`playbook`。这是用于Ansible 配置文件的最常用位置。在实践中，很少会使用`~/.ansible.cfg`或`/etc/ansible/ansible.cfg`文件。     
&#8195;&#8195;可以通过将不同的配置文件放在不同的目录中，然后从适当目录执行Ansible命令，以此利用配置文件；随着配置文件数量的增加，这种方法难以管理。可以通过`ANSIBLE_CONFIG`环境变量定义配置文件的位置，Ansible将使用变量所指定的配置文件，而不用上面提及的任何配置文件。
### 配置文件优先级
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
### 管理配置文件中的设置
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

### 配置连接
&#8195;&#8195;Ansible需要知道如何与其受管主机通信。更改配置文件的一个最常见原因是为了控制Ansible使用什么方法和用户来管理受管主机。需要的一些信息包括：
- 列出受管主机和主机组的清单的位置
- 要使用哪一种连接协议来与受管主机通信(默认为SSH)，以及是否需要非标准网络端口来连接服务器
- 要在受管主机上使用哪一远程用户；可以是root用户或者某一非特权用户
- 如果远程用户为非特权用户，Ansible需要知道它是否应尝试将特权升级为root以及如何进行升级(如使用sudo)
- 是否提示输入SSH密码或sudo密码以进行登录或获取特权

#### 清单位置
&#8195;&#8195;在`[defaults]`部分中，`inventory`指令可以直接指向某一静态清单文件，或者指向含有多个静态清单文件和动态清单脚本的某一目录。
#### 连接设置
连接设置：
- 默认情况下，Ansible使用SSH协议连接受管主机。控制Ansible如何连接受管主机的最重要参数在`[defaults]`部分中设置。
- 默认情况下，Ansible尝试连接受管主机时使用的用户名与运行Ansible命令的本地用户相同。若要指定不同的远程用户，将`remote_user`参数设置为该用户名
- 如果为运行Ansible的本地用户配置了SSH私钥，使得它们能够在受管主机上进行远程用户的身份验证，则 Ansible将自动登录。如果不是这种情况，可以通过设置指令`ask_pass = true`，将Ansible配置为提示本地用户输入由远程用户使用的密码
- 如果在使用一个Linux控制节点，并对受管主机使用OpenSSH，如果可以使用密码以远程用户身份登录，那么或许可以设置基于SSH密钥的身份验证，从而能够设置`ask_pass = false`
- 第一步是确保在`~/.ssh`中为控制节点上的用户配置了SSH密钥对。可以运行`ssh-keygen`命令来实现这个目标

#### 升级特权
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
```ini
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
### 非SSH连接
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
### 配置文件注释
Ansible配置文件允许使用两种注释字符：井号或编号符号(#)以及分号(;)。
- 位于行开头的编号符号会注释掉整行。它不能和指令位于同一行中
- 分号字符可以注释掉所在行中其右侧的所有内容。它可以和指令位于同一行中，只要该指令在其左侧

### 示例练习
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
```ini
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
## 运行临时命令
### 使用Ansible运行临时命令
#### 运行临时命令
使用`ansible`命令来运行临时命令：
```
ansible host-pattern -m module [-a 'module arguments'] [-i inventory]
```
参数说明：
- `host-pattern`参数用于指定应在其上运行临时命令的受管主机：
    - 可以是清单中的特定受管主机或主机组
    - 可以与`--list-hosts`选项结合使用，此选项显示通过特定主机模式匹配的主机
    - 可以使用`-i`选项来指定要使用的其他清单位置，取代默认位置
- `-m`选项将Ansible应在目标主机上运行的`module`的名称作为参数：
    - 模块是为了实施任务而执行的小程序
    - 一些模块不需要额外的信息，但其他模块需要使用额外参数来指定其操作详情
    - `-a`选项以带引号字符串形式取这些参数的列表

&#8195;&#8195;一种最简单的临时命令是使用`ping`模块。此模块不执行`ICMP ping`，而是检查能否在受管主机上运行基于Python的模块。示例临时命令确定清单中的所有受管主机能否运行标准的模块：
```
[ansible@redhat9 ~]$ ansible all -m ping
192.168.100.131 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
redhat8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```
#### 通过模块来执行任务
&#8195;&#8195;模块是临时命令用于完成任务的工具。Ansible提供了数百个能够完成不同任务的模块。命令`ansible-doc -l`可列出系统上安装的所有模块。可以使用`ansible-doc`来按照名称查看特定模块的文档，再查找关于模块将取什么参数作为选项的信息。示例显示`ping`模块的文档：
```
[ansible@redhat9 ~]$ ansible-doc ping
> ANSIBLE.BUILTIN.PING    (/usr/lib/python3.9/site-packages/ansible/modules/pin>
...output omitted...
ADDED IN: historical
OPTIONS (= is mandatory):
- data
        Data to return for the `ping' return value.
        If this parameter is set to `crash', the module will cause an
        exception.
        [Default: pong]
        type: str
...output omitted...
```
Ansible模块各种类型：
- 文件模块：
    - `copy`：将本地文件复制到受管主机
    - `file`：设置文件的权限和其他属性
    - `lineinfile`：确保特定行是否在文件中
    - `synchronize`：使用rsync同步内容
- 软件包模块：
    - `package`：使用操作系统本机的自动检测软件包管理器管理软件包
    - `yum`：使用YUM软件包管理器管理软件包
    - `apt`：使用APT软件包管理器管理软件包
    - `dnf`：使用DNF软件包管理器管理软件包
    - `gem`：管理Ruby gem
    - `pip`：从PyPI管理Python软件包
- 系统模块：
    - `firewalld`：使用firewalld管理任意端口和服务
    - `reboot`：重新启动计算机
    - `service`：管理服务
    - `user`：添加、删除和管理用户帐户
- Net Tools模块
    - `get_url`：通过HTTP、HTTPS或FTP下载文件
    - `nmcli`：管理网络
    - `uri`：与Web服务交互

大部分模块会取用参数。可在模块的文档中找到可用于该模块的参数列表：
- 临时命令可以通过`-a`选项向模块传递参数
- 无需参数时，可以从临时命令中省略`-a`选项
- 如果需要指定多个参数，以引号括起的空格分隔列表形式提供

示例临时命令使用`user`模块来确保`huang`用户存在于`redhat8`上并且其UID为`1000`：
```
[ansible@redhat9 ~]$ ansible -m user -a 'name=huang uid=1000 state=present' redhat8
redhat8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "append": false,
    "changed": false,
    "comment": "huang",
    "group": 1000,
    "home": "/home/huang",
    "move_home": false,
    "name": "huang",
    "shell": "/bin/bash",
    "state": "present",
    "uid": 1000
}
```
&#8195;&#8195;大多数模块为`idempotent`，这表示它们可以安全地多次运行；如果系统已处于正确的状态，它们不会进行任何操作。例如再次运行前面的临时命令，应该不会报告任何更改。
#### command模块
&#8195;&#8195;`command`模块允许管理员在受管主机的命令行中运行任意命令。运行的命令通过`-a`选项指定为该模块的参数。示例对由`testhosts]`主机模式引用的受管主机运行`hostname`命令：
```
[ansible@redhat9 ~]$ ansible testhosts -m command -a /usr/bin/hostname
192.168.100.131 | CHANGED | rc=0 >>
redhat8
redhat8 | CHANGED | rc=0 >>
redhat8
```
示例临时命令为每个受管主机返回两行输出：
- 第一行是状态报告，显示对其运行该临时操作的受管主机名称及操作的结果
- 第二行是使用Ansible command模块远程执行的命令的输出

使用`-o`选项以单行格式显示Ansible临时命令的输出：
```
[ansible@redhat9 ~]$ ansible testhosts -m command -a /usr/bin/hostname -o
192.168.100.131 | CHANGED | rc=0 | (stdout) redhat8
redhat8 | CHANGED | rc=0 | (stdout) redhat8
```
`command`模块注意事项：
- `command`模块对受管主机执行的远程命令不是由受管主机上的`shell`处理。因此无法访问`shell`环境变量，也不能执行重定向和传送等`shell`操作
- 如果临时命令没有指定哪个模块与`-m`选项一起使用，Ansible将默认使用`command`模块

#### shell模块
&#8195;&#8195;在命令需要shell处理的情形中，可以使用`shell`模块。与`command`模块类似，可以在临时命令中将要执行的命令作为参数传递给该模块：
- 与`command`模块不同的是，这些命令将通过受管主机上的shell进行处理。因此，可以访问shell环境变量，也可使用重定向和传送等shell操作

&#8195;&#8195;示例`command`模块和`shell`模块的区别。尝试使用这两个模块执行内建的Bash命令`set`，只有使用`shell`模块时才会成功：
```
[ansible@redhat9 ~]$ ansible redhat8 -m command -a set
redhat8 | FAILED | rc=2 >>
[Errno 2] No such file or directory: b'set': b'set'
[ansible@redhat9 ~]$ ansible redhat8 -m shell -a set
redhat8 | CHANGED | rc=0 >>
BASH=/bin/sh
...output omitted...
```
#### raw模块
&#8195;&#8195;`command`和`shell`模块都要求受管主机上安装正常工作的Python。第三个模块是`raw`，可以绕过模块子系统，直接使用远程shell运行命令：
- 在管理无法安装Python的系统(如网络路由器)时，可以利用`raw`模块
- 也可用于将Python安装到主机上

#### 三个模块注意事项
在大多数情形中，建议避免使用`command`、`shell`和`raw`这三个`运行命令`模块：
- 其他模块大部分都是幂等的，可以自动进行更改跟踪：
    - 可以用于测试系统的状态，在这些系统已处于正确的状态时不执行任何操作
    - 相反，以幂等方式使用`运行命令`模块要复杂得多。依靠它们，更难以确信再次运行临时命令或playbook 不会造成意外的失败
    - 当`shell`或`command`模块运行时，通常会基于它是否认为影响了计算机状态而报告`CHANGED`状态
- 有时候`运行命令`模块也是解决问题的好办法。如需使用，建议最好先尝试使用 `command`模块，只有在需要`shell`或`raw`模块的特殊功能时才利用它们

### 配置临时命令的连接
&#8195;&#8195;受管主机连接和特权升级的指令可以在Ansible配置文件中配置，也可使用临时命令中的选项来定义。使用临时命令中的选项定义时，它们将优先于Ansible配置文件中配置的指令。下表显示了与各项配置文件指令类同的命令行选项。

配置文件指令|命令行选项
:---|:---
inventory|-i
remote_user|-u
become|--become, -b
become_method|--become-method
become_user|--become-user
become_ask_pass|--ask-become-pass, -K

可以通过查询`ansible --help`的输出来确定其当前定义的值：
```
[ansible@redhat9 ~]$ ansible --help
...output omitted...
  -c CONNECTION, --connection CONNECTION
                        connection type to use (default=smart)
  -u REMOTE_USER, --user REMOTE_USER
                        connect as this user (default=None)
```
Ansible模块文档：[Ansible Module](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html)
## 编写和运行Playbook
### Ansible Playbook说明
&#8195;&#8195;`任务`是指应用模块来执行特定工作单元。 `play`是一系列任务，按顺序应用于从清单中选择的一台或多台主机。`playbook`是一个文本文件，其中包含由一个或多个按特定顺序运行的`play`组成的列表。
- `Play`可以让用户将一系列冗长而复杂的手动管理任务转变为可轻松重复的例程，并且具有可预测的成功成果
- 在`playbook`中，您可以将`play`内的任务序列保存为人类可读并可立即运行的形式
- 根据任务的编写方式，任务本身记录了部署应用或基础架构所需的步骤

### 格式化Ansible Playbook
&#8195;&#8195;临时命令`ansible -m user -a 'name=huang uid=1000 state=present' redhat8`可以重新编写为一个单任务`play`并保存在`playbook`中。生成的`playbook`如下：
```yaml
---
- name: Configure important user consistently
  hosts: redhat8
  tasks:
    - name: huang exists with UID 1000
      user:
        name: huang
        uid: 1000
        state: present
```
&#8195;&#8195;`Playbook`是以YAML格式编写的文本文件，通常使用扩展名`yml`保存。`Playbook`使用空格字符(不允许使用制表符)缩进来表示其数据结构。YAML对用于缩进的空格数量没有严格的要求，但有两个基本的规则：
- 处于层次结构中同一级别的数据元素(例如同一列表中的项目)必须具有相同的缩进量
- 如果项目属于其他项目的子项，其缩进量必须大于父项

&#8195;&#8195;`Playbook`开头的一行由三个破折号`---`组成，这是文档开始标记。其末尾可能使用三个圆点`...`作为文档结束标记，通常会省略。在这两个标记之间，会以一个`play`列表的形式来定义`playbook`。YAML列表中的项目以一个破折号加空格开头。示例：
```yaml
- Thor
- America Captain
- Thanos
```
&#8195;&#8195;`Play`本身是一个键值对集合。同一`play`中的键应当使用相同的缩进量。下面示例显示了具有三个键的YAML代码片段。前两个键具有简单的值。第三个将含有三个项目的列表作为值。示例：
```yaml
  Company: Marvel
  Alliance: The Avengers
  superhero:
    - Thor
    - America Captain
    - Iron Man
```
生成的`playbook`详细说明：
- 原始示例`play`有三个键：`name`、`hosts`和`tasks`，这些键都有相同的缩进
- `play`的第一行开头是破折号加空格(表示该play是列表中的第一项)，而后是第一个键，即`name`属性：
    - `name`键将一个任意字符串作为标签与该`play`关联。示例中标识了该`play`的用途：Configure important user consistently
    - `name`键是可选的，但建议使用，有助于记录`playbook`。在`playbook`中包含多个`play`时特别有用
- `Play`中的第二个键是`hosts`属性，指定对其运行`play`中的任务的主机。与用于`ansible`命令的参数相似，`hosts`属性将主机模式取为值，如清单中受管主机或组的名称
- `Play`中的最后一个键是`tasks`属性，其值指定要为该`play`运行的任务的列表。本例中只有一项任务，该任务使用特定参数运行`user`模块(以确保用户`huang`存在并且具有UID`1000`)
- 作为`play`中的一部分，`tasks`属性按顺序实际列出要在受管主机上运行的任务。列表中各项任务本身是一个键值对集合。示例中，`play`中的唯一任务含有两个键：
    - `name`是记录任务用途的可选标签。建议命名所有任务，从而帮助记录自动流程中每一步的用途
    - `user`是要为这个任务运行的模块。其参数作为一组键值对传递，它们是模块的子项(`name`、`uid`和 `state`)

&#8195;&#8195;`playbook`中`play`和任务列出的顺序很重要，Ansible会按照顺序运行它们。下面示例为含有多项任务的`tasks`属性的示例，该示例使用`service`模块确保为多个网络服务启用在引导时启动：
```yaml
  tasks:
    - name: web server is enabled
      service:
        name: httpd
        enabled: true

    - name: NTP server is enabled
      service:
        name: chronyd
        enabled: true

    - name: Postfix is enabled
      service:
        name: postfix
        enabled: true
```
### 运行Playbook
&#8195;&#8195;`ansible-playbook`命令可用于运行`playbook`。该命令在控制节点上执行，要运行的`playbook`的名称则作为参数传递。示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook test1.yml

PLAY [Configure important user consistently] ***********************************

TASK [Gathering Facts] *********************************************************
ok: [redhat8]

TASK [huang exists with UID 1000] **********************************************
ok: [redhat8]

PLAY RECAP *********************************************************************
redhat8                    : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
运行`playbook`说明：
- 在运行`playbook`时，将生成输出来显示所执行的`play`和任务。输出中也会报告执行的每一项任务的结果
- 在运行`playbook`时，屏幕中会显示每个`play`和任务的`name`键的值
- `Gathering Facts`任务是一项特别的任务，`setup`模块通常在`play`启动时自动运行这项任务
- 对于含有多个`play`和任务的`playbook`，设置`name`属性后可以更加轻松地监控`playbook`执行的进展
- 通常而言，`Ansible playbook`中的任务是幂等的，而且能够安全地多次运行`playbook`。如果目标受管主机已处于正确的状态，则不应进行任何更改

#### 提高输出的详细程度
&#8195;&#8195;`ansible-playbook`命令提供的默认输出不提供详细的任务执行信息。`ansible-playbook -v`命令提供了额外的信息，总共有四个级别。具体说明如下表：

选项|描述
:---|:---
-v|显示任务结果
-vv|任务结果和任务配置都会显示
-vvv|包含关于与受管主机连接的信息
-vvvv|增加了连接插件相关的额外详细程度选项，包括受管主机上用于执行脚本的用户，以及所执行的脚本

#### 语法验证
&#8195;&#8195;在执行`playbook`之前，最好要进行验证，确保其内容的语法正确无误。`ansible-playbook`命令中 `--syntax-check`选项可用于验证`playbook`的语法。验证成功示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook --syntax-check  test1.yml

playbook: test1.yml
```
#### 执行空运行
&#8195;&#8195;使用`-C`选项对`playbook`执行空运行。这会使Ansible报告在执行该`playbook`时将会发生什么更改，但不会对受管主机进行任何实际的更改。示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook -C site.yml
PLAY [Install and start Apache HTTPD] ******************************************************************************

TASK [Gathering Facts] ******************************************************************************
ok: [redhat8]

TASK [httpd package is present] *******************************************************************************
ok: [redhat8]

TASK [httpd is started] *******************************************************************************
changed: [redhat8]

PLAY RECAP ********************************************************************
redhat8                    : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
&#8195;&#8195;演示了一个`playbook`的空运行，它包含单项任务，可确保在受管主机上安装了最新版本的 `httpd`软件包。该空运行报告此任务会对受管主机产生的更改。
## 实施多个Play
### 编写多个Play
&#8195;&#8195;`Playbook`是一个YAML文件，由一个或多个`play`组成的列表。`play`按顺序列出了要对清单中的选定主机执行的任务。因此，如果一个`playbook`中含有多个`play`，每个`play`可以将其任务应用到单独的一组主机。
- 在编排可能涉及对不同主机执行不同任务的复杂部署时很方便。可以对一组主机运行一个`play`，完成后再对另一组主机运行另一个`play`
- `Playbook`中的各个`play`编写为`playbook`中的顶级列表项。各个`play`是含有常用`play`关键字的列表项

&#8195;&#8195;示例含有两个`play`的简单`playbook`。第一个`play`针对`redhat8`运行，第二个`play`则针对`192.168.100.131`运行：
```yaml
---
# This is a simple playbook with two plays
- name: first play
  hosts: redhat8
  tasks:
    - name: first task
      yum:
        name: httpd
        status: present

    - name: second task
      service:
        name: httpd
        enabled: true

- name: second play
  hosts: 192.168.100.131
  tasks:
    - name: first task
      service:
        name: sshd
        enabled: true
```
### Play中的远程用户和特权升级
&#8195;&#8195;`Play`可以将不同的远程用户或特权升级设置用于`play`，取代配置文件中指定的默认设置。这些都在`play`本身中与`hosts`或`tasks`关键字相同的级别上设置。
#### 用户属性
&#8195;&#8195;`Playbook`中的任务与临时命令相同，用于任务执行的用户帐户取决于`Ansible`配置文件`/etc/ansible/ansible.cfg`中的不同关键字。运行任务的用户可以通过`remote_user`关键字来定义。如果启用了特权升级，`become_user`等其他关键字也会发生作用。可以在`play`中使用`remote_user`关键字来覆盖`Ansible`配置中定义的远程用户：
```yaml
remote_user: remoteuser
```
#### 特权升级属性
&#8195;&#8195;可以提供额外的关键字，从而在`playbook`内定义特权升级参数。`become`布尔值关键字可用于启用或禁用特权升级，无论它在`Ansible`配置文件中的定义为何。可以取`yes`或`true`值来启用特权升级，或者取`no`或`false`值来禁用它：
```yaml
become: true
```
&#8195;&#8195;如果启用了特权升级，则可以使用`become_method`关键字来定义特定`play`期间要使用的特权升级方法。示例指定`sudo`用于特权升级：
```yaml
become_method: sudo
```
&#8195;&#8195;启用了特权升级时，`become_user`关键字可定义特定`play`上下文内要用于特权升级的用户帐户。示例如下：
```yaml
become_user: privileged_user
```
示例在`play`中使用这些关键字：
```yaml
- name: /etc/hosts is up to date
  hosts: redhat8
  remote_user: ansible
  become: yes

  tasks:
    - name: redhat9 in /etc/hosts
      lineinfile:
        path: /etc/hosts
        line: '192.168.100.133 redhat9 server'
        state: present
```
### 查找用于任务的模块
#### 模块文档
&#8195;&#8195;Ansible随附打包的大量模块为管理员提供了许多用于常见管理任务的工具。使用`ansible-doc`命令来查找关于本地系统上安装的模块的信息。使用`ansible-doc -l`命令查看控制节点上可用模块的列表，将显示模块名称列表以及其功能的概要。示例如下：
```
[ansible@redhat9 ~]$ ansible-doc -l
apt                                            Manages apt-packages
apt_key                                        Add or remove an apt key
apt_repository                                 Add and remove APT repositories
assemble                                       Assemble configuration files from fragments
...output omitted...
```
&#8195;&#8195;使用`ansible-doc [模块名称]`命令来显示模块的详细文档。与Ansible文档网站一样，该命令提供模块功能的概要、其不同选项的详细信息，以及示例。命令示例如下：
```
[ansible@redhat9 ~]$ ansible-doc git
> ANSIBLE.BUILTIN.GIT    (/usr/lib/python3.9/site-packages/ansible/modules/git.py)
        Manage `git' checkouts of repositories to deploy files or software.

ADDED IN: version 0.0.1 of ansible-core
OPTIONS (= is mandatory):

- accept_hostkey
        If `yes', ensure that "-o StrictHostKeyChecking=no" is present as an ssh option.
        [Default: no]
        type: bool
        added in: version 1.5 of ansible-core
...output omitted...
```
&#8195;&#8195;`ansible-doc`命令的`-s`选项会生成示例输出，可以充当如何在`playbook`中使用特定模块的示范。输出中包含的注释，提醒管理员各个选项的用法。示例如下：
```
[ansible@redhat9 ~]$ ansible-doc -s git
- name: Deploy software (or files) from git checkouts
  git:
      accept_hostkey:        # If `yes', ......
      accept_newhostkey:     # As of OpenSSH 7.5, ......
      archive:               # Specify archive file path with extension......
      archive_prefix:        # Specify a prefix to add to each file path......
      bare:                  # If `yes', repository will be created ......
      clone:                 # If `no', do not clone the repository even ......
...output omitted...
```
#### 模块维护
&#8195;&#8195;`Ansible`随附了大量模块，它们可用于执行许多任务。这在该模块的 `ansible-doc`输出末尾的`METADATA`部分中指明上游`Ansible `社区中维护该模块的人。`status`字段记录模块的开发状态：
- `stableinterface`：模块的关键字稳定，将尽力确保不删除关键字或更改其含义
- `preview`：模块处于技术预览阶段，可能不稳定，其关键字可能会更改，或者它可能需要本身会受到不兼容更改的库或Web服务
- `deprecated`：模块已被弃用，未来某一发行版中将不再提供
- `removed`：模块已从发行版中移除，但因文档需要存在存根，以帮助之前的用户迁移到新的模块

`supported_by`字段记录上游Ansible社区中维护该模块的人。可能的值包括：
- `core`：由上游核心Ansible开发人员维护，始终随Ansible提供
- `curated`：模块由社区中的合作伙伴或公司提交并维护
- `community`：模块不受到核心上游开发人员、合作伙伴或公司的支持，完全由一般开源社区维护

&#8195;&#8195;可以自己编写私有的模块，或者从第三方获取模块。Ansible会在`ANSIBLE_LIBRARY`环境变量指定的位置查找自定义模块；未设置此变量时由当前Ansible配置文件中的`library`关键字指定。Ansible也在相对于当前运行的`playbook`的`./library`目录中搜索自定义模块。示例：
```ini
library = /home/ansible/my_modules
```
相关文档参考：
- 有关编写方法的文档链接：[Ansible Modules](https://docs.ansible.com/ansible/2.9/dev_guide/developing_modules.html)
- 上游Ansible社区有用于Ansible的问题跟踪器及其集成模块，链接为：[https://github.com/ansible/ansible/issues](https://github.com/ansible/ansible/issues)

#### 注意事项
&#8195;&#8195;使用`ansible-doc`命令可以查找和了解如何为用户的任务使用模块。`command`、`shell`和`raw`模块的用法可能看似简单，应尽量避免在`playbook`中使用它们。因为它们可以取用任意命令，因此使用这些模块时很容易写出非幂等的`playbook`。示例如下：
```yaml
- name: Non-idempotent approach with shell module
  shell: echo "nameserver 192.168.100.1" > /etc/resolv.conf
```
示例说明；
- 示例为使用`shell`模块的任务为非幂等
- 每次运行`play`时，它都会重写`/etc/resolv.conf`，即使它已经包含了行`nameserver 192.168.100.1`

&#8195;&#8195;可以通过多种方式编写以幂等方式使用`shell`模块的任务，而且有时候进行这些更改并使用 `shell`是最佳的做法。使用`copy`模块是更快的方案。示例如果`/etc/resolv.conf`文件已包含正确的内容，则不会重写该文件：
```yaml
- name: Idempotent approach with copy module
  copy:
    dest: /etc/resolv.conf
    content: "nameserver 192.168.100.1\n"
```
&#8195;&#8195;`copy`模块可以测试来了解是否达到了需要的状态，如果已达到，则不进行任何更改。`shell`模块容许非常大的灵活性，但需要注意，确保它以幂等方式运行。幂等的`playbook`可以重复运行，确保系统处于特定的状态，而不会破坏状态已经正确的系统。
### Playbook语法变化
#### YAML注释
&#8195;&#8195;注释可以用于提高可读性。在YAML中，编号或井号符号`#`右侧的所有内容都是注释。如果注释的左侧有内容，请在该编号符号的前面加一个空格。示例：
```yaml
# This is a YAML comment
- Name: Thor # This is also a YAML comment
```
#### YAML字符串
&#8195;&#8195;YAML中的字符串通常不需要放在引号里，即使字符串中包含空格。字符串可以用双引号或单引号括起。三个示例分别如下：
```yaml
this is a string
'this is another string'
"this is yet another a string"
```
编写多行字符串一种方式是使用竖线`|`字符表示要保留字符串中的换行字符：
```yaml
include_newlines: |
        Example Company
        123 Main Street
        Atlanta, GA 30303
```
&#8195;&#8195;编写多行字符串，也可以使用大于号`>`字符来表示换行字符转换成空格并且行内的引导空白将被删除。这种方法通常用于将很长的字符串在空格字符处断行，使它们跨占多行来提高可读性。示例：
```yaml
fold_newlines: >
        This is an example
        of a long string,
        that will become
        a single sentence once folded.
```
#### YAML字典
以缩进块的形式编写的键值对集合，示例：
```yaml
  name: America Captain
  from: New York
  power: 9999
```
字典也可以使用以花括号括起的内联块格式编写，示例：
```yaml
 { name: Wonder Woman,from: Amazon,power: 9998}
```
#### YAML列表
使用普通单破折号语法编写的列表：
```yaml
  hosts:
    - hosta
    - hostb
    - hostc
```
也有以方括号括起的内联格式：
```yaml
superhero: [Wonder Woman, Amazon, 9998]
```
#### 过时的“键=值”Playbook简写
&#8195;&#8195;某些`playbook`可能使用较旧的简写方法，通过将模块的键值对放在与模块名称相同的行上来定义任务。示例：
```yaml
  tasks:
    - name: shorthand form
      service: name=httpd enabled=true state=started
```
通常避免简写，使用下面格式：
```yaml
  tasks:
    - name: normal form
      service:
        name: httpd
        enabled: true
        state: started
```