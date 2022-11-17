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