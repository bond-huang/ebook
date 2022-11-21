# Ansible-利用角色简化Playbook
## 描述角色结构
### 利用角色构造Ansible playbook
&#8195;&#8195;随着开发更多的playbook，可能会发现有很多机会重复利用以前编写的playbook中的代码.Ansible角色提供了一种方法，能以通用的方式更加轻松地重复利用Ansible代码。可以在标准化目录结构中打包所有任务、变量、文件、模板，以及调配基础架构或部署应用所需的其他资源。只需通过复制相关的目录，将角色从一个项目复制到另一个项目。然后，只需从一个play调用该角色就能执行它。    
&#8195;&#8195;借助编写良好的角色，可以从playbook中向角色传递调整其行为的变量，设置所有站点相关的主机名、IP地址、用户名，或其他在本地需要的具体详细信息。例如，部署数据库服务器的角色可能已编写为支持多个变量，这些变量用于设置主机名、数据库管理员用户和密码，以及需要为安装进行自定义的其他参数。角色的作者也可以确保在选择不在play中设置变量值时，为这些变量设定合理的默认值。

Ansible角色具有下列优点：
- 角色可以分组内容，从而与他人轻松共享代码
- 可以编写角色来定义系统类型的基本要素：Web服务器、数据库服务器、Git存储库，或满足其他用途
- 角色使得较大型项目更容易管理
- 角色可以由不同的管理员并行开发

&#8195;&#8195;除了自行编写、使用、重用和共享角色外，也可以从其他来源获取角色。一些角色已包含在 `rhel-system-roles`软件包中，作为RHEL的一部分。也可以从Ansible Galaxy网站获取由社区提供支持的许多角色。
### 检查Ansible角色结构
&#8195;&#8195;Ansible角色由子目录和文件的标准化结构定义。顶级目录定义角色本身的名称。文件整理到子目录中，子目录按照各个文件在角色中的用途进行命名，如`tasks`和`handlers`。`files`和`templates`子目录中包含由其他`YAML`文件中的任务引用的文件。官方示例`tree`命令显示`user.example`角色的目录结构：
```
[user@host roles]$ tree user.example
user.example/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```
Ansible角色子目录：

子目录|功能
:---:|:---
defaults|此目录中的`main.yml`文件包含角色变量的默认值，使用角色时可以覆盖这些默认值。这些变量的优先级较低，应该在play中更改和自定义
files|此目录包含由角色任务引用的静态文件
handlers|此目录中的`main.yml`文件包含角色的处理程序定义
meta|此目录中的`main.yml`文件包含与角色相关的信息，如作者、许可证、平台和可选的角色依赖项
tasks|此目录中的`main.yml`文件包含角色的任务定义
templates|此目录包含由角色任务引用的Jinja2模板
tests|此目录可以包含清单和`test.yml` playbook，可用于测试角色
VAR|此目录中的`main.yml`文件定义角色的变量值。这些变量通常用于角色内部用途。这些变量的优先级较高，在playbook中使用时不应更改

### 定义变量和默认值
&#8195;&#8195;角色变量通过在角色目录层次结构中创建含有键值对的`vars/main.yml`文件来定义。与其他变量一样，这些角色变量在角色YAML文件中引用：`{{ VAR_NAME }}`。这些变量具有较高的优先级，无法被清单变量覆盖。这些变量旨在供角色的内部功能使用：
- 默认变量允许为可在play中使用的变量设置默认值，以配置角色或自定义其行为
- 它们通过在角色目录层次结构中创建含有键值对的`defaults/main.yml`文件来定义
- 默认变量具有任何可用变量中最低的优先级。它们很容易被包括清单变量在内的任何其他变量覆盖
- 这些变量旨在让人们在编写使用该角色的play时可以准确地自定义或控制它将要执行的操作
- 它们可用于向角色提供所需的信息，以正确地配置或部署某些对象
- 在`vars/main.yml`或`defaults/main.yml`中定义具体的变量，但不要在两者中都定义。有意要覆盖变量的值时，应使用默认变量

注意事项：
- 角色不应该包含特定于站点的数据。它们绝对不应包含任何机密，如密码或私钥
- 这是因为角色应该是通用的、可以重复利用并自由共享。特定于站点的详细信息不应硬编码到角色中
- 机密应当通过其他途径提供给角色。这是可能要在调用角色时设置角色变量的一个原因。Play中设置的角色变量可以提供机密，或指向含有该机密的Ansible Vault加密文件

### 在Playbook中使用Ansible角色
示例调用Ansible角色的一种方式：
```yml
---
- hosts: remote.example.com
  roles:
    - role1
    - role2
```
&#8195;&#8195;对于每个指定的角色，角色任务、角色处理程序、角色变量和角色依赖项将按照顺序导入到 playbook中。角色中的任何`copy`、`script`、`template`或`include_tasks/import_tasks`任务都可引用角色中相关的文件、模板或任务文件，且无需相对或绝对路径名称。Ansible将分别在角色的`files`、`templates`或`tasks`子目录中寻找它们。    
&#8195;&#8195;如果使用`roles`部分将角色导入到play中，这些角色会在为该play定义的任何任务之前运行。示例设置`role2`的两个角色变量`var1`和`var2`的值。使用`role2`时，任何`defaults`和`vars`变量都会被覆盖：
```yml
---
- hosts: remote.example.com
  roles:
    - role: role1
    - role: role2
      var1: val1
      var2: val2
```
以下是在此情形中可能会看到的另一种等效的YAML语法：
```yml
---
- hosts: remote.example.com
  roles:
    - role: role1
    - { role: role2, var1: val1, var2: val2 }
```
注意事项：
- 正如前面的示例中所示，内嵌设置的角色变量（角色参数）具有非常高的优先级。它们将覆盖大多数其他变量
- 务必要谨慎，不要重复使用内嵌设置在play中任何其他位置的任何角色变量的名称，因为角色变量的值将覆盖清单变量和任何play vars

### 控制执行顺序
&#8195;&#8195;对于playbook中的每个play，任务按照任务列表中的顺序来执行。执行完所有任务后，将执行任何通知的处理程序。
- 在角色添加到play中后，角色任务将添加到任务列表的开头。如果play中包含第二个角色，其任务列表添加到第一个角色之后
- 角色处理程序添加到play中的方式与角色任务添加到play中相同。每个play定义一个处理程序列表。角色处理程序先添加到处理程序列表，后跟play的`handlers`部分中定义的任何处理程序
- 在某些情形中，可能需要在角色之前执行一些play任务。若要支持这样的情形，可以为play配置`pre_tasks`部分。列在此部分中的所有任务将在执行任何角色之前执行。如果这些任务中有任何一个通知了处理程序，则这些处理程序任务也在角色或普通任务之前执行
- play也支持`post_tasks`关键字。这些任务在play的普通任务和它们通知的任何处理程序运行之后执行

&#8195;&#8195;以下play演示了一个带有`pre_tasks`、`roles`、`tasks`、`post_tasks`和`handlers`的示例。一个play中通常不会同时包含所有这些部分：
```yml
- name: Play to illustrate order of execution
  hosts: remote.example.com
  pre_tasks:
    - debug:
        msg: 'pre-task'
      notify: my handler
  roles:
    - role1
  tasks:
    - debug:
        msg: 'first task'
      notify: my handler
  post_tasks:
    - debug:
        msg: 'post-task'
      notify: my handler
  handlers:
    - name: my handler
      debug:
        msg: Running my handler
```
上例中，每个部分中都执行`debug`任务来通知`my handler`处理程序。`my handler`任务执行了三次：
- 在执行了所有`pre_tasks`任务后
- 在执行了所有角色任务和`tasks`部分中的任务后
- 在执行了所有`post_tasks`后

&#8195;&#8195;除了将角色包含在play的`roles`部分中外，也可以使用普通任务将角色添加到play中。使用 `include_role`模块可以动态包含角色，使用`import_role`模块则可静态导入角色。示例如何通过 `include_role`模块来利用任务包含角色：
```yml
- name: Execute a role as a task
  hosts: remote.example.com
  tasks:
    - name: A normal task
      debug:
        msg: 'first task'
    - name: A task to include role2 here
      include_role: role2
```
注意：
- `include_role`模块是在Ansible 2.3中新增的
- `import_role`模块则是Ansible 2.4中新增的

## 利用系统角色重用内容
### RHEL系统角色
&#8195;&#8195;自RHEL 7.4开始，操作系统随附了多个Ansible角色，作为`rhel-system-roles`软件包的一部分。在RHEL 8中，该软件包可从`AppStream`频道获取。以下是每个角色的简要描述：

名称|状态|角色描述
:---:|:---:|:---
rhel-system-roles.kdump|全面支持|配置kdump崩溃恢复服务
rhel-system-roles.network|全面支持|配置网络接口
rhel-system-roles.selinux|全面支持|配置和管理SELinux自定义，包括SELinux模式、文件和端口上下文、布尔值设置，以及SELinux用户
rhel-system-roles.timesync|全面支持|使用网络时间协议或精确时间协议配置时间同步
rhel-system-roles.postfix|技术预览|使用Postfix服务将每个主机配置为邮件传输代理
rhel-system-roles.firewall|开发中|配置主机的防火墙
rhel-system-roles.tuned|开发中|配置tuned服务，以调优系统性能

&#8195;&#8195;系统角色的目的是在多个版本之间标准化RHEL子系统的配置。使用系统角色来配置版本6.10及以上的任何红帽企业Linux。
#### 简化配置管理
&#8195;&#8195;举例而言，RHEL 7的建议时间同步服务为`chronyd`服务。但在RHEL 6 中，建议的服务为`ntpd`服务。在混合了RHEL 6和7主机的环境中，管理员必须管理这两个服务的配置文件：
- 借助RHEL系统角色，管理员不再需要维护这两个服务的配置文件
- 管理员可以使用`rhel-system-roles.timesync`角色来配置RHEL 6和7主机的时间同步
- 一个包含角色变量的简化YAML文件可以为这两种类型的主机定义时间同步配置

#### 支持RHEL系统角色
&#8195;&#8195;RHEL系统角色衍生自Ansible Galaxy上的开源Linux系统角色项目。与Linux系统角色不同，RHEL系统角色是作为标准红帽企业Linux订阅的一部分而受到红帽的支持。RHEL系统角色具有红帽企业Linux订阅附带的相同生命周期支持优势。其他角色正在上游Linux系统角色项目中开发，但尚未通过RHEL订阅提供，这些角色可通过Ansible Galaxy获取。
### 安装RHEL系统角色
&#8195;&#8195;RHEL系统角色由`rhel-system-roles`软件包提供，该软件包可从`AppStream`频道获取。在 Ansible控制节点上安装该软件包。
#### 安装RHEL系统角色
&#8195;&#8195;示例采用`yum`程序安装`rhel-system-roles`软件包。该程序假定控制节点已注册到红帽企业Linux订阅，并已安装Ansible：
```
[root@redhat9 ~]# yum install rhel-system-roles
```
安装的RHEL9已经默认安装了，RHEL系统角色在目录`/usr/share/ansible/roles`下：
```
[ansible@redhat9 ~]$ ls /usr/share/ansible/roles
linux-system-roles.certificate  linux-system-roles.ssh      rhel-system-roles.metrics
linux-system-roles.cockpit      linux-system-roles.sshd     rhel-system-roles.nbde_client
...output omitted... 
```
&#8195;&#8195;每个角色的对应上游名称都链接到RHEL系统角色。这使角色可在playbook中通过任一名称来引用。RHEL中的默认`roles_path`在路径中包含`/usr/share/ansible/roles`，在playbook引用这些角色时Ansible会自动找到它们。注意：
- 如果`roles_path`在当前Ansible配置文件中已被覆盖，并且已设置`ANSIBLE_ROLES_PATH`环境变量，或者`roles_path`中更早列出的目录下存在另一个同名角色，则Ansible可能无法找到系统角色。

#### 访问RHEL系统角色的文档
&#8195;&#8195;安装后，RHEL系统角色的文档位于`/usr/share/doc/rhel-system-roles/`目录中。文档按照子系统整理到子目录中：
```
[ansible@redhat9 ~]$ ls -l /usr/share/doc/rhel-system-roles
total 20
drwxr-xr-x. 2 root root 4096 Sep  4 13:56 certificate
drwxr-xr-x. 2 root root   42 Sep  4 13:56 cockpit
drwxr-xr-x. 3 root root   55 Sep  4 13:56 collection
drwxr-xr-x. 2 root root   42 Sep  4 13:56 crypto_policies
drwxr-xr-x. 2 root root   42 Sep  4 13:56 firewall
...output omitted... 
```
系统角色的文档说明：
- 每个角色的文档目录均包含一个`README.md`文件。`README.md`文件含有角色的说明，以及角色用法信息
- `README.md`文件也会说明影响角色行为的角色变量。通常，`README.md`文件中含有一个playbook代码片段，用于演示常见配置场景的变量设置
- 部分角色文档目录中含有示例playbook。首次使用某一角色时，请查看文档目录中的任何额外示例playbook
- RHEL系统角色的角色文档与Linux系统角色的文档相匹配。可以访问Ansible Galaxy网站[https://galaxy.ansible.com](https://galaxy.ansible.com)上的上游角色的角色文档。

### 时间同步角色示例
&#8195;&#8195;需要在服务器上配置NTP时间同步，可以自行编写自动化来执行每一个必要的任务。RHEL系统角色中有一个可以执行此操作的角色，那就是`rhel-system-roles.timesync`：
- 该角色记录在`/usr/share/doc/rhel-system-roles/timesync`目录中的`README.md`文件中。此文件说明了影响角色行为的所有变量，还包含演示了不同时间同步配置的三个playbook代码片段
- 为了手动配置NTP服务器，该角色有一个`timesync_ntp_servers`变量。此变量取一个要使用的NTP服务器的列表作为值。列表中的每一项均由一个或多个属性构成

`timesync_ntp_servers`两个关键属性如下：

属性|用途
:---:|:---
主机名称|要与其同步的 NTP 服务器的主机名
iburst|一个布尔值，用于启用或禁用快速初始同步。在角色中默认为`no`，但通常应该将该属性设为`yes`

&#8195;&#8195;下面示例play使用`rhel-system-roles.timesync`角色将受管主机配置为利用快速初始同步从三个NTP服务器获取时间。此外，还添加了一个任务，以使用`timezone`模块将主机的时区设为`UTC`：
```yml
- name: Time Synchronization Play
  hosts: servers
  vars:
    timesync_ntp_servers:
      - hostname: 0.rhel.pool.ntp.org
        iburst: yes
      - hostname: 1.rhel.pool.ntp.org
        iburst: yes
      - hostname: 2.rhel.pool.ntp.org
        iburst: yes
    timezone: UTC

  roles:
    - rhel-system-roles.timesync

  tasks:
    - name: Set timezone
      timezone:
        name: "{{ timezone }}"
```
示例说明及注意事项：
- 此示例在play的`vars`部分中设置角色变量，但更好的做法可能是将它们配置为主机或主机组的清单变量
- 如果要设置不同的时区，可以使用`tzselect`命令查询其他有效的值。也可以使用`timedatectl`命令来检查当前的时钟设置

例如一个playbook项目具有以下结构：
```
[root@host playbook-project]# tree
.
├── ansible.cfg
├── group_vars
│   └── servers
│       └── timesync.yml1
├── inventory
└── timesync_playbook.yml2
```
&#8195;&#8195;示例中`timesync.yml1`定义时间同步变量，覆盖清单中`servers`组内主机的角色默认值。此文件内容对比之前示例，类似于：
```yml
timesync_ntp_servers:
  - hostname: 0.rhel.pool.ntp.org
    iburst: yes
  - hostname: 1.rhel.pool.ntp.org
    iburst: yes
  - hostname: 2.rhel.pool.ntp.org
    iburst: yes
timezone: UTC
```
示例中`timesync_playbook.yml2`的Playbook内容简化为：
```yml
- name: Time Synchronization Play
  hosts: servers
  roles:
    - rhel-system-roles.timesync
  tasks:
    - name: Set timezone
      timezone:
        name: "{{ timezone }}"
```
示例结构说明：
- 该结构可清楚地分隔角色、playbook代码和配置设置。Playbook代码简单易读，应该不需要复杂的重构。角色内容由红帽进行维护并提供支持。所有设置都以清单变量的形式进行处理
- 该结构还支持动态的异构环境。具有新的时间同步要求的主机可能会放置到新的主机组中。相应的变量在YAML文件中定义，并放置到相应的`group_vars`（或`host_vars`）子目录中

### SELinux角色示例
&#8195;&#8195;`rhel-system-roles.selinux`角色可以简化SELinux配置设置的管理。它通过利用SELinux相关的Ansible模块来实施。与自行编写任务相比，使用此角色的优势是它能让用户摆脱编写这些任务的职责。取而代之，用户将为角色提供变量以对其进行配置，且角色中维护的代码将确保应用用户需要的SELinux配置。此角色可以执行的任务包括：
- 设置`enforcing`或`permissive`模式
- 对文件系统层次结构的各部分运行`restorecon`
- 设置SELinux布尔值
- 永久设置SELinux文件上下文
- 设置SELinux用户映射

#### 调用SELinux角色
&#8195;&#8195;有时候，SELinux角色必须确保重新引导受管主机，以便能够完整应用其更改。但是，它本身从不会重新引导主机。如此一来，便可以控制重新引导的处理方式。不过，这意味着在play中正确使用此角色要比平常复杂一些。     
&#8195;&#8195;其工作方式为，该角色将一个布尔值变量`selinux_reboot_required`设为`true`，如果需要重新引导，则失败。可以使用`block/rescue`结构来从失败中恢复，具体操作为：如果该变量未设为`true`，则让play失败，如果值是`true`，则重新引导受管主机并重新运行该角色。Play中的块应该类似于：
```yml
    - name: Apply SELinux role
      block:
        - include_role:
            name: rhel-system-roles.selinux
      rescue:
        - name: Check for failure for other reasons than required reboot
          fail:
          when: not selinux_reboot_required

        - name: Restart managed host
          reboot:

        - name: Reapply SELinux role to complete changes
          include_role:
            name: rhel-system-roles.selinux
```
#### 配置SELinux角色
&#8195;&#8195;用于配置`rhel-system-roles.selinux`角色的变量的详细记录位于其`README.md`文件中。下面示例演示了使用此角色的一些方法。   
&#8195;&#8195;`selinux_state`变量设置`SELinux`的运行模式。它可以设为`enforcing`、`permissive`或`disabled`。如果未设置，则不更改模式：
```yml
selinux_state: enforcing
```
&#8195;&#8195;`selinux_booleans`变量取一个要调整的SELinux布尔值的列表作为值。列表中的每一项是变量的散列/字典：布尔值 名称、状态（应为`on`或`off`），以及该设置在重新引导后是否应永久有效。本例将 `httpd_enable_homedirs`永久设为`on`：
```yml
selinux_booleans:
  - name: 'httpd_enable_homedirs'
    state: 'on'
    persistent: 'yes'
```
&#8195;&#8195;`selinux_fcontext`变量取一个要永久设置（或删除）的文件上下文的列表作为值。它的工作方式与`selinux fcontext`命令非常相似。下面示例确保策略中包含一条规则，用于将`/srv/www`下所有文件的默认SELinux类型设为`httpd_sys_content_t`：
```yml
selinux_fcontexts:
  - target: '/srv/www(/.*)?'
    setype: 'httpd_sys_content_t'
    state: 'present'
```
`selinux_restore_dirs`变量指定要对其运行`restorecon`的目录的列表：
```yml
selinux_restore_dirs:
  - /srv/www
```
`selinux_ports`变量取应当具有特定SELinux类型的端口的列表作为值：
```yml
selinux_ports:
  - ports: '82'
    setype: 'http_port_t'
    proto: 'tcp'
    state: 'present'
```
## 创建角色
### 角色创建流程
在Ansible中创建角色不需要特别的开发工具。创建和使用角色包含三个步骤：
- 创建角色目录结构
- 定义角色内容
- 在playbook中使用角色

### 创建角色目录结构
&#8195;&#8195;Ansible默认会在Ansible Playbook所在目录的roles子目录中查找角色。这样可以利用playbook和其他支持文件存储角色。如果Ansible无法在该位置找到角色，它会按照顺序在Ansible配置设置`roles_path`所指定的目录中查找。此变量包含要搜索的目录的冒号分隔列表。此变量的默认值为：
```
~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
```
&#8195;&#8195;这允许用户将角色安装到由多个项目共享的系统上。例如，可能将自己的角色安装在自己的主目录下的`~/.ansible/roles`子目录中，而系统可能将所有用户的角色安装在`/usr/share/ansible/roles`目录中。每个角色具有自己的目录，采用标准化的目录结构。例如，以下目录结构包含了定义`motd`角色的文件：
```
[user@host ~]$ tree roles/
roles/
└── motd
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    ├── meta
    │   └── main.yml
    ├── README.md
    ├── tasks
    │   └── main.yml
    └── templates
        └── motd.j2
```
目录结构说明：
- `README.md`提供人类可读的基本角色描述、有关如何使用该角色的文档和示例，以及其发挥作用所需要满足的任何非Ansible要求
- `meta`子目录包含一个`main.yml`文件，该文件指定有关模块的作者、许可证、兼容性和依赖项的信息
- `files`子目录包含固定内容的文件
- `templates`子目录则包含使用时可由角色部署的模板
- 其他子目录中可以包含`main.yml`文件，它们定义默认的变量值、处理程序、任务、角色元数据或变量，具体取决于所处的子目录
- 如果某一子目录存在但为空，如示例中的`handlers`，它将被忽略
- 如果某一角色不使用功能，则其子目录可以完全省略。例如，示例中的`vars`子目录已被省略

#### 创建角色框架
&#8195;&#8195;可以使用标准Linux命令创建新角色所需的所有子目录和文件。也可以通过命令行实用程序来自动执行新角色创建过程：
- `ansible-galaxy`命令行工具可用于管理Ansible角色，包括新角色的创建
- 可以运行`ansible-galaxy init`来创建新角色的目录结构。指定角色的名称作为命令的参数，该命令在当前工作目录中为新角色创建子目录

创建示例如下：
```
[ansible@redhat9 ~]$ pwd
/home/ansible
[ansible@redhat9 ~]$ cd playbook-project/
[ansible@redhat9 playbook-project]$ cd roles
[ansible@redhat9 roles]$ ansible-galaxy init my_new_role
- Role my_new_role was created successfully
[ansible@redhat9 roles]$ ls my_new_role/
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```
### 定义角色内容
&#8195;&#8195;创建目录结构后，必须编写角色的内容。可以从`ROLENAME/tasks/main.yml`任务文件开始，该文件是由角色运行的主要任务列表。    
&#8195;&#8195;下列`tasks/main.yml`文件管理着受管主机上的`/etc/motd`文件。它使用`template`模块将名为`motd.j2`的模板部署到受管主机上。因为`template`模块是在角色任务而非playbook任务内配置的，所以从角色的`templates`子目录中检索`motd.j2`模板：
```
[user@host ~]$ cat roles/motd/tasks/main.yml
---
# tasks file for motd
- name: deliver motd file
  template:
    src: motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: 0444
```
下面是`motd`角色的`motd.j2`模板的内容。它引用了Ansible事实和`system_owner`变量：
```
[user@host ~]$ cat roles/motd/templates/motd.j2
This is the system {{ ansible_facts['hostname'] }}.

Today's date is: {{ ansible_facts['date_time']['date'] }}.

Only use this system with permission.
You can ask {{ system_owner }} for access.
```
&#8195;&#8195;该角色为`system_owner`变量定义一个默认值。角色目录结构中的`defaults/main.yml`文件就是设置这个值的位置。    
&#8195;&#8195;下列`defaults/main.yml`文件将`system_owner`变量设置为`user@host.example.com`。此电子邮件地址将写入到该角色所应用的受管主机上的`/etc/motd`文件中：
```
[user@host ~]$ cat roles/motd/defaults/main.yml
---
system_owner: user@host.example.com
```
#### 角色内容开发的推荐做法
&#8195;&#8195;角色允许以模块化方式编写playbook。为了最大限度地提高新开发角色的效率，在角色开发中采用以下推荐做法：
- 在角色自己的版本控制存储库中维护每个角色。Ansible很适合使用基于git的存储库
- 角色存储库中不应存储敏感信息，如密码或SSH密钥。敏感值应以变量的形式进行参数化，其默认值应不敏感。使用角色的Playbook负责通过`Ansible Vault`变量文件、环境变量或其他`ansible-playbook`选项定义敏感变量
- 使用`ansible-galaxy init`启动用户的角色，然后删除不需要的任何目录和文件
- 创建并维护`README.md`和`meta/main.yml`文件，以记录角色的用途、作者和用法
- 让角色侧重于特定的用途或功能。可以编写多个角色，而不是让一个角色承担许多任务
- 经常重用和重构角色。避免为边缘配置创建新的角色。如果现有角色能够完成大部分的所需配置，请重构现有角色以集成新的配置方案。使用集成和回归测试技术来确保角色提供所需的新功能，并且不对现有的playbook造成问题

### 定义角色依赖项
&#8195;&#8195;角色依赖项使得角色可以将其他角色作为依赖项包含在内。例如一个定义文档服务器的角色可能依赖于另一个安装和配置Web服务器的角色。依赖关系在角色目录层次结构中的`meta/main.yml`文件内定义。示例如下：
```yml
---
dependencies:
  - role: apache
    port: 8080
  - role: postgres
    dbname: serverlist
    admin_user: felix
```
&#8195;&#8195;默认情况下，角色仅作为依赖项添加到playbook中一次。若有其他角色也将它作为依赖项列出，它不会再次运行。此行为可以被覆盖，将`meta/main.yml`文件中的`allow_duplicates`变量设置为`yes`即可。
### 在Playbook中使用角色
&#8195;&#8195;要访问角色，可在play的`roles:`部分引用它。下列playbook引用了`motd`角色。由于没有指定变量，因此将使用默认变量值应用该角色：
```yml
---
- name: use motd role playbook
  hosts: remote.example.com
  remote_user: devops
  become: true
  roles:
    - motd
```
&#8195;&#8195;执行该playbook时，因为角色而执行的任务可以通过角色名称前缀来加以识别。以下示例输出通过任务名称中的`motd :`前缀进行了演示(假定`motd`角色位于`roles`目录中)：
```
[user@host ~]$ ansible-playbook -i inventory use-motd-role.yml

PLAY [use motd role playbook] **************************************************

TASK [setup] *******************************************************************
ok: [remote.example.com]

TASK [motd: deliver motd file] ************************************************
changed: [remote.example.com]

PLAY RECAP *********************************************************************
remote.example.com         : ok=2    changed=1    unreachable=0    failed=0
```
#### 通过变量更改角色的行为
&#8195;&#8195;编写良好的角色利用默认变量来改变角色行为，使之与相关的配置场景相符。这有助于让角色变得更为通用，可在各种不同的上下文中重复利用。如果通过以下方式定义了相同的变量，则角色的`defaults`目录中定义的变量的值将被覆盖：
- 在清单文件中定义，作为主机变量或组变量
- 在playbook项目的`group_vars`或`host_vars`目录下的YAML文件中定义
- 作为变量嵌套在play的`vars`关键字中定义
- 在play的`roles`关键字中包含该角色时作为变量定义

&#8195;&#8195;示例如何将`motd`角色与`system_owner`角色变量的不同值搭配使用。角色应用到受管主机时，指定的值`someone@host.example.com`将取代变量引用：
```yml
---
- name: use motd role playbook
  hosts: remote.example.com
  remote_user: devops
  become: true
  vars:
    system_owner: someone@host.example.com
  roles:
    - role: motd
```
&#8195;&#8195;以这种方式定义时，`system_owner`变量将替换同一名称的默认变量的值。嵌套在`vars`关键字内的任何变量定义不会替换在角色的`vars`目录中定义的同一变量的值。    
&#8195;&#8195;下例演示如何将`motd`角色与`system_owner`角色变量的不同值搭配使用。指定的值 `someone@host.example.com`将替换变量引用，不论是在角色的`vars`还是`defaults`目录中定义：
```yml
---
- name: use motd role playbook
  hosts: remote.example.com
  remote_user: devops
  become: true
  roles:
    - role: motd
      system_owner: someone@host.example.com
```
注意，在play中使用角色变量时，变量的优先顺序可能会让人困惑：
- 几乎任何其他变量都会覆盖角色的默认变量，如清单变量、`play vars`变量，以及内嵌的角色参数等
- 较少的变量可以覆盖角色的`vars`目录中定义的变量。事实、通过`include_vars`加载的变量、注册的变量和角色参数是其中一些具备这种能力的变量。清单变量和`play vars`无此能力。这非常重要，因为它有助于避免play意外改变角色的内部功能
- 不过，正如上述示例中最后一个所示，作为角色参数内嵌声明的变量具有非常高的优先级。它们可以覆盖角色的 `vars`目录中定义的变量。如果某一角色参数的名称与`play vars`或角色`vars`中设置的变量或者清单变量或 playbook变量的名称相同，该角色参数将覆盖另一个变量

## 使用Ansible Galaxy部署角色
### Ansible Galaxy介绍
&#8195;&#8195;Ansible Galaxy是一个Ansible内容公共资源库，这些内容由许多Ansible管理员和用户编写。它包含数千个Ansible角色，具有可搜索的数据库，可帮助Ansible用户确定或许有助于他们完成管理任务的角色。Ansible Galaxy含有面向新的Ansible用户和角色开发人员的文档和视频链接。官方网站：[https://galaxy.ansible.com/](https://galaxy.ansible.com/)

### Ansible Galaxy命令行工具
&#8195;&#8195;`ansible-galaxy`命令行工具可以用于搜索角色，显示角色相关的信息，以及安装、列举、删除或初始化角色。
#### 从命令行搜索角色
&#8195;&#8195;`ansible-galaxy search`子命令在Ansible Galaxy中搜索角色。如果以参数形式指定了字符串，则可用于按照关键字在Ansible Galaxy中搜索角色。可以使用`--author`、`--platforms`和 `--galaxy-tags`选项来缩小搜索结果的范围。也可以将这些选项用作主要的搜索键。    
&#8195;&#8195;例如，命令 `ansible-galaxy search --author geerlingguy`将显示由用户`geerlingguy`提交的所有角色。   
&#8195;&#8195;结果按照字母顺序显示，而不是`Best Match`分数降序排列。下例显示了包含`redis`并且适用于企业Linux(EL)平台的角色的名称：
```
[user@host ~]$ ansible-galaxy search 'redis' --platforms EL
Found 124 roles matching your search:
 Name                                  Description
 ----                                  -----------
...output omitted...
 geerlingguy.php-redis                 PhpRedis support for Linux
 geerlingguy.redis                     Redis for Linux
 gikoluo.filebeat                      Filebeat for Linux.
...output omitted...
```
&#8195;&#8195;`ansible-galaxy info`子命令显示与角色相关的更多详细信息。Ansible Galaxy从多个位置获取这一信息，包括角色的`meta/main.yml`文件及其GitHub存储库。以下命令显示了Ansible Galaxy提供的 `geerlingguy.redis`角色的相关信息：
```
[user@host ~]$ ansible-galaxy info geerlingguy.redis
Role: geerlingguy.redis
        description: Redis for Linux
        active: True
...output omitted...
        license: license (BSD, MIT)
        min_ansible_version: 2.4
        modified: 2018-11-19T14:53:29.722718Z
        open_issues_count: 11
        path: [u'/etc/ansible/roles', u'/usr/share/ansible/roles']
        role_type: ANS
        stargazers_count: 98
...output omitted...
```
#### 从Ansible Galaxy安装角色
&#8195;&#8195;`ansible-galaxy install`子命令从Ansible Galaxy下载角色，并将它安装到控制节点本地。默认情况下，角色安装到用户的`roles_path`下的第一个可写目录中。根据为Ansible设置的默认`roles_path`，角色通常将安装到用户的`~/.ansible/roles`目录：
- 默认的`roles_path`可能会被当前Ansible配置文件或环境变量`ANSIBLE_ROLES_PATH`覆盖，这将影响`ansible-galaxy`的行为
- 也可以通过使用`-p DIRECTORY`选项，指定具体的目录来安装角色。

&#8195;&#8195;下例中，`ansible-galaxy`将`geerlingguy.redis`角色安装到playbook项目的roles目录中。命令的当前工作目录是`/opt/project`：
```
[user@host project]$ ansible-galaxy install geerlingguy.redis -p roles/
- downloading role 'redis', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/...output omitted...
- extracting geerlingguy.redis to /opt/project/roles/geerlingguy.redis
- geerlingguy.redis (1.6.0) was installed successfully
[user@host project]$ ls roles/
geerlingguy.redis
```
#### 使用要求文件安装角色
&#8195;&#8195;也可以使用`ansible-galaxy`，根据某一文本文件中的定义来安装一个角色列表。例如，如果一个playbook需要安装特定的角色，可以在项目目录中创建一个`roles/requirements.yml`文件来指定所需的角色。此文件充当playbook项目的依赖项清单，使得playbook的开发和测试能与任何支持角色分开进行。

例如，一个用于安装`geerlingguy.redis`的简单`requirements.yml`可能类似于如下：
```yml
- src: geerlingguy.redis
  version: "1.5.0"
```
示例说明：
- `src`属性指定角色的来源，本例中为来自Ansible Galaxy的`geerlingguy.redis`角色
- `version`属性是可选的，指定要安装的角色版本，本例中为`1.5.0`

若要使用角色文件来安装角色，可使用`-r REQUIREMENTS-FILE`选项：
```
[user@host project]$ ansible-galaxy install -r roles/requirements.yml \
> -p roles
- downloading role 'redis', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-redis/archive/1.6.0.tar.gz
- extracting geerlingguy.redis to /opt/project/roles/geerlingguy.redis
- geerlingguy.redis (1.6.0) was installed successfully
```
&#8195;&#8195;可以使用`ansible-galaxy`来安装不在Ansible Galaxy中的角色。可以在私有的Git存储库或Web服务器上托管自有的专用或内部角色。下例演示了如何利用各种远程来源配置要求文件：
```
[user@host project]$ cat roles/requirements.yml
# from Ansible Galaxy, using the latest version
- src: geerlingguy.redis

# from Ansible Galaxy, overriding the name and using a specific version
- src: geerlingguy.redis
  version: "1.5.0"
  name: redis_prod

# from any Git-based repository, using HTTPS
- src: https://gitlab.com/guardianproject-ops/ansible-nginx-acme.git
  scm: git
  version: 56e00a54
  name: nginx-acme

# from any Git-based repository, using SSH
- src: git@gitlab.com:guardianproject-ops/ansible-nginx-acme.git
  scm: git
  version: master
  name: nginx-acme-ssh

# from a role tar ball, given a URL;
#   supports 'http', 'https', or 'file' protocols
- src: file:///opt/local/roles/myrole.tar
  name: myrole
```
示例说明：
- `src`关键字指定Ansible Galaxy角色名称。如果角色没有托管在Ansible Galaxy中，则`src`关键字将指明角色的URL
- 如果角色托管在来源控制存储库中，则需要使用`scm`属性。`ansible-galaxy`命令能够从基于Git或`Mercurial`的软件存储库下载和安装角色：
    - 基于Git 的存储库要求`scm`值为`git`，而托管在`Mercurial`存储库中的角色则要求值为`hg`
    - 如果角色托管在Ansible Galaxy中，或者以`tar`存档形式托管在Web服务器上，则省略`scm`关键字
- `name`关键字用于覆盖角色的本地名称
- `version`关键字用于指定角色的版本。`version`关键字可以是与来自角色的软件存储库的分支、标记或提交哈希对应的任何值

若要安装与playbook项目关联的角色，可执行`ansible-galaxy install`命令：
```
[user@host project]$ ansible-galaxy install -r roles/requirements.yml \
> -p roles
- downloading role 'redis', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-redis/archive/1.6.0.tar.gz
- extracting geerlingguy.redis to /opt/project/roles/geerlingguy.redis
- geerlingguy.redis (1.6.0) was installed successfully
- downloading role 'redis', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-redis/archive/1.5.0.tar.gz
- extracting redis_prod to /opt/project/roles/redis_prod
- redis_prod (1.5.0) was installed successfully
- extracting nginx-acme to /opt/project/roles/nginx-acme
- nginx-acme (56e00a54) was installed successfully
- extracting nginx-acme-ssh to /opt/project/roles/nginx-acme-ssh
- nginx-acme-ssh (master) was installed successfully
- downloading role from file:///opt/local/roles/myrole.tar
- extracting myrole to /opt/project/roles/myrole
- myrole was installed successfully
```
#### 管理下载的角色
&#8195;&#8195;`ansible-galaxy`命令也可管理本地的角色，如位于playbook项目的`roles`目录中的角色。`ansible-galaxy list`子命令列出本地找到的角色：
```
[user@host project]$ ansible-galaxy list
- geerlingguy.redis, 1.6.0
- myrole, (unknown version)
- nginx-acme, 56e00a54
- nginx-acme-ssh, master
- redis_prod, 1.5.0
```
可以使用`ansible-galaxy remove`子命令本地删除角色：
```
[user@host ~]$ ansible-galaxy remove nginx-acme-ssh
- successfully removed nginx-acme-ssh
[user@host ~]$ ansible-galaxy list
- geerlingguy.redis, 1.6.0
- myrole, (unknown version)
- nginx-acme, 56e00a54
- redis_prod, 1.5.0
```
&#8195;&#8195;在playbook中使用下载并安装的角色的方式与任何其他角色都一样。在`roles`部分中利用其下载的角色名称来加以引用。如果角色不在项目的`roles`目录中，则将检查`roles_path`来查看角色是否安装在了其中一个目录中，将使用第一个匹配项。以下`use-role.yml` playbook引用了`redis_prod`和`geerlingguy.redis`角色：
```
[user@host project]$ cat use-role.yml
---
- name: use redis_prod for Prod machines
  hosts: redis_prod_servers
  remote_user: devops
  become: true
  roles:
    - redis_prod

- name: use geerlingguy.redis for Dev machines
  hosts: redis_dev_servers
  remote_user: devops
  become: true
  roles:
    - geerlingguy.redis
```
&#8195;&#8195;此playbook使不同版本的`geerlingguy.redis`角色应用到生产和开发服务器。借助这种方式，可以对角色更改进行系统化测试和集成，然后再部署到生产服务器上。如果角色的近期更改造成了问题，则借助版本控制来开发角色，就能回滚到过去某一个稳定的角色版本。
## 从内容集合获取角色和模块
### 讨论内容集合
&#8195;&#8195;Ansible内容集合是Ansible内容的一种分发格式。该集合会提供一组相关模块、角色和插件，您可以将它们下载到控制节点，然后在Playbook中使用。例如：
- `redhat.insights`集合对模块和角色进行分组，可以使用这些模块和角色来在RHEL的红帽`Insights`上注册一个系统
- `cisco.ios`集合对管理Cisco IOS网络设备的模块和插件进行分组。该集合由Cisco公司负责支持和维护
- `community.crypto`集合提供用于创建`SSL/TLS`证书的模块

&#8195;&#8195;内容集合支持将核心Ansible代码与模块和插件分开更新。这样便于供应商和开发人员按照自己的节奏维护和分发集合，不受Ansible版本的影响。也可以自行开发集合，为团队自定义角色和模块。    
&#8195;&#8195;内容集合合还可提供更大的灵活性。通过使用集合，可以仅安装所需内容，而不必安装所有受支持的模块。还可以选择集合的特定版本（可以是早期版本或晚期版本），或者在红帽或供应商支持的集合版本或社区提供的集合版本之间进行选择。
#### 在命名空间中组织集合
&#8195;&#8195;为了便于按名称指定集合及其内容，命名空间中还会整理集合名称。供应商、合作伙伴、开发人员和内容创建者可使用命名空间为其集合分配唯一名称，而不与其他开发人员发生冲突。    
&#8195;&#8195;命名空间是集合名称的第一部分。例如，由Ansible社区维护的所有集合可能会放入`community`命名空间中，名称类似于`community.crypto`、`community.postgresql`和`community.rabbitmq`。由红帽维护和支持的集合则可能会放入redhat命名空间，名称类似于`redhat.rhv`、`redhat.satellite`和`redhat.insights`。
#### 选择集合来源
Ansible提供以下两个下载和安装集合的官方来源：
- Ansible自动化中心
    - Ansible自动化中心主要托管红帽及其合作伙伴为客户提供支持时会使用的Ansible内容集合。
    - 红帽会审核、维护并更新这些集合，并提供全面支持。例如，该平台会提供`redhat.rhv`、`redhat.satellite`、 `redhat.insights`和`cisco.ios`集合
    - 需要有效订阅红帽Ansible自动化平台服务才能访问Ansible自动化中心。可使用[https://cloud.redhat.com/ansible/automation-hub/](https://cloud.redhat.com/ansible/automation-hub/)上的 Ansible自动化中心Web UI来列出和访问集合
- Ansible Galaxy
    - Ansible Galaxy主要托管各类Ansible开发人员和用户提交的集合
    - Ansible Galaxy是一个公共库，不提供官方支持，但允许公开访问。例如，该平台会提供`community.crypto`、`community.postgresql`和`community.rabbitmq`集合
    - 可使用[https://galaxy.ansible.com/](https://galaxy.ansible.com/)上的Ansible Galaxy Web UI来搜索集合

### 安装内容集合
&#8195;&#8195;要想让playbook使用集合中的内容，必须在控制节点上安装相应集合。使用`ansible-galaxy`命令从多个可能来源下载集合，包括Ansible Galaxy。示例使用`ansible-galaxy`命令和`collection`参数下载`community.crypto`集合并在本地系统安装的情况：
```
[ansible@redhat9 ~]$ ansible-galaxy collection install community.crypto
```
该命令还可用于从本地或远程tar存档安装集合：
```
[user@controlnode ~]$ ansible-galaxy collection install \
> /tmp/community-dns-1.2.0.tar.gz
[user@controlnode ~]$ ansible-galaxy collection install \
> http://www.example.com/redhat-insights-1.0.5.tar.gz
```
&#8195;&#8195;Ansible配置指令`collection_paths`可指定一个用冒号分隔的列表，其中列出了Ansible在系统中寻找已安装集合的路径。可以在`ansible.cfg`配置文件中设置此指令。`ansible-galaxy`命令默认会将集合安装到`Collections_paths`指令定义的第一个目录中：
- `collections_paths`的默认值为`~/.ansible/collections:/usr/share/ansible/collections`。因此，`ansible-galaxy`命令默认会将集合安装在`~/.ansible/collections`目录中

如果要将集合安装到其他目录，使用`--collections-path`（或 -p）选项。
```
[root@controlnode ~]# ansible-galaxy collection install \
> -p /usr/share/ansible/collections community.postgresql
```
&#8195;&#8195;使用`--collections-path`（或 -p）选项时，请确保选择`collections_paths`指令中列出的目录。`ansible-playbook`命令也会使用该指令来查找集合。如果不使用`collections_paths`指令中定义的路径，playbook将无法找到所安装的集合。
#### 使用要求文件来安装集合
&#8195;&#8195;可以创建`requirements.yml`文件，列出需要安装的所有集合。在Ansible项目中添加`collections/requirements.yml`文件后，团队成员便可立即确定所需集合。此外，在运行playbook之前，自动化控制器还会检测该文件并自动安装所需集合。    
&#8195;&#8195;以下`requirements.yml`文件列出了多个要安装的集合。注意，可以指定特定集合版本，也可以提供本地或远程tar存档：
```yml
---
collections:
  - name: community.crypto

  - name: ansible.posix
    version: 1.2.0

  - name: /tmp/community-dns-1.2.0.tar.gz

  - name: http://www.example.com/redhat-insights-1.0.5.tar.gz
```
&#8195;&#8195;然后，`ansible-galaxy`命令可以使用该文件来安装所有集合。使用`--requirements-file`（或 -r）选项向该命令提供`requirements.yml`文件：
```
[root@controlnode ~]# ansible-galaxy collection install -r requirements.yml
```
#### 配置集合来源
&#8195;&#8195;`ansible-galaxy`命令默认会使用[https://galaxy.ansible.com/](https://galaxy.ansible.com/)位置的Ansible Galaxy来下载集合。要让该命令同时使用Ansible自动化中心来下载集合，将以下指令添加到`ansible.cfg`文件中：
```ini
...output omitted...
[galaxy]
server_list = automation_hub, galaxy

[galaxy_server.automation_hub]
url=https://cloud.redhat.com/api/automation-hub/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token 
token=eyJh...Jf0o

[galaxy_server.galaxy]
url=https://galaxy.ansible.com/
```
配置说明：
- `server_list`：按顺序列出`ansible-galaxy`命令必须使用的所有存储库：
    - 对于所定义的每个名称，请添加 `[galaxy_server.name]`部分以提供连接参数
    - 鉴于Ansible自动化中心可能无法提供playbook需要的所有集合，也可以将Ansible Galaxy添加到最后一个位置作为备用，如果该集合在Ansible自动化中心不可用，则`ansible-galaxy`命令将在Ansible Galaxy 中检索
- `url`：提供用于访问存储库的URL
- `auth_url`：提供用于身份验证的URL
- `token`：访问Ansible自动化中心，需要有一个与帐户关联的身份验证令牌。可使用Ansible自动化中心Web UI来生成该令牌

除了令牌外，也可通过`username`和`password`参数提供客户门户网站的用户名和密码进行访问：
```ini
...output omitted...
[galaxy_server.automation_hub]
url=https://cloud.redhat.com/api/automation-hub/
username=operator1
password=Sup3r53cR3t
...output omitted...
```
&#8195;&#8195;可能不希望在`ansible.cfg`文件中公开凭证，因为该文件可能会因版本控制而被提交。遇到这种情况，从 `ansible.cfg`文件中删除身份验证参数，并将其定义为环境变量。可按照以下方式定义环境变量：
```
ANSIBLE_GALAXY_SERVER_<server_id>_<key>=value
```
示例说明：
- `server_id`：大写的服务器标识符。服务器标识符是一个名称，在`server_list`参数中以及`[galaxy_server.server_id]`部分的名称中会用到
- `key`：大写的参数名称

以下示例提供token参数作为环境变量：
```
[user@controlnode ~]$ cat ansible.cfg
...output omitted...
[galaxy_server.automation_hub]
url=https://cloud.redhat.com/api/automation-hub/
auth_url=https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token
[user@controlnode ~]$ export \
> ANSIBLE_GALAXY_SERVER_AUTOMATION_HUB_TOKEN='eyJh...Jf0o'
[user@controlnode ~]$ ansible-galaxy collection install ansible.posix
```
### 使用集合
&#8195;&#8195;安装集合后，可以使用临时命令和playbook来使用该集合。从Ansible自动化中心或Ansible Galaxy Web UI访问集合文档，以检索关于所提供的角色和模块的信息。也可以检查系统上的集合目录结构。该集合会将模块存储在`plugins/modules/`目录中，将角色存储在`roles/`目录中：
```
[user@controlnode ~]$ tree \
> ~/.ansible/collections/ansible_collections/redhat/insights/
/home/user/.ansible/collections/ansible_collections/redhat/insights/
...output omitted...
├── plugins
│   ├── action
│   │   └── insights_config.py
│   ├── inventory
│   │   └── insights.py
│   └── modules
│       ├── insights_config.py
│       └── insights_register.py
...output omitted...
├── roles
│   ├── compliance
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   ├── install.yml
│   │   │   ├── main.yml
│   │   │   └── run.yml
│   │   └── tests
│   │       ├── compliance.yml
│   │       ├── install-only.yml
│   │       └── run-only.yml
│   └── insights_client
...output omitted...
```
&#8195;&#8195;要使用某个模块或角色，请使用其完全限定的集合名称(FQCN)进行引用。根据前面的输出内容，所引用的`redhat.insights.insights_client`角色为`insights_client`。示例临时命令将从`community.general`集合调用`mail`模块：
```
[user@controlnode ~]$ ansible localhost -m community.general.mail \
> -a 'subject="Hello World" to=root'
```
示例playbook将从`community.mysql`集合调用`mysql_user`模块：
```yml
---
- name: Create the operator1 user in the test database
  hosts: db.example.com

  tasks:
    - name: Ensure the operator1 database user is defined
      community.mysql.mysql_user:
        name: operator1
        password: Secret0451
        priv: '.:ALL'
        state: present
```
示例playbook将从`redhat.satellite`集合调用`organizations`角色：
```yml
---
- name: Add the test organizations to Red Hat Satellite
  hosts: localhost

  tasks:
    - name: Ensure the organizations exist
      include_role:
        name: redhat.satellite.organizations
      vars:
        satellite_server_url: https://sat.example.com
        satellite_username: admin
        satellite_password: Sup3r53cr3t
        satellite_organizations:
          - name: test1
            label: tst1
            state: present
            description: Test organization 1
          - name: test2
            label: tst2
            state: present
            description: Test organization 2
```
#### 使用Ansible 2.9之后的Ansible内置集合
&#8195;&#8195;在Ansible的未来版本中，核心安装始终会包含一个名为`ansible.buildin`的特殊集合。此集合将会包含一组常见模块，如`copy`、`template`、`file`、`yum`、`command`和`service`：
- 始终可以在playbook中使用这些模块的短名称。例如，仍然可以直接使用`file`，而不用使用`ansible.buildin.file`
- 这样一来，很多Ansible 2.9 playbook无需修改即可运行。但对于`ansible.builderin`中未包含模块的其他集合，仍需先安装才能使用
- 红帽仍建议您使用`FQCN`表示法，以防未来与可能的同名集合发生冲突

示例playbook使用`FQCN`表示法来寻找`yum`、`copy`和`service`模块：
```yml
---
- name: Install and start Apache HTTPD
  hosts: web

  tasks:
    - name: Ensure the httpd package is present
      ansible.builtin.yum:
        name: httpd
        state: present

    - name: Ensure the index.html file is present
      ansible.builtin.copy:
        src: files/index.html
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: 0644
        setype: httpd_sys_content_t

    - name: Ensure the httpd service is started
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true
```
## 练习