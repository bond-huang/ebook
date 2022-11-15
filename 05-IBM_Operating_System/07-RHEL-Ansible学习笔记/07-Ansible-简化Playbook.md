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