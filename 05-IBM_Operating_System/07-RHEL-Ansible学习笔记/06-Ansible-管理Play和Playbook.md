# Ansible-管理Play和Playbook
## 利用主机模式选择主机
### 引用清单主机
&#8195;&#8195;`主机模式`用于指定要作为play或临时命令的目标的主机。在最简单的形式中，清单中受管主机或主机组的名称就是指定该主机或主机组的主机模式：
- 在play中，hosts指定要针对其运行play的受管主机
- 对于临时命令，以命令行参数形式将主机模式提供给ansible命令

&#8195;&#8195;仔细地使用主机模式并设有适当的清单组，而不在play的任务中设置复杂的条件，通常更容易控制作为play目标的主机。学习环境跟官方环境不一致，以官方inventory示例进行学习记录：
```ini
web.example.com
data.example.com

[lab]
labhost1.example.com
labhost2.example.com

[test]
test1.example.com
test2.example.com

[datacenter1]
labhost1.example.com
test1.example.com

[datacenter2]
labhost2.example.com
test2.example.com

[datacenter:children]
datacenter1
datacenter2

[new]
192.168.2.1
192.168.2.2
```
&#8195;&#8195;如何解析主机模式，执行`ansible-playbook playbook.yml`，使用不同的主机模式来以此示例清单中受管主机的不同子集作为目标。
#### 受管主机
&#8195;&#8195;最基本的主机模式是单一受管主机名称列在清单中。这将指定该主机是清单中 ansible命令要执行操作的唯一主机：
- 在该playbook运行时，第一个`Gathering Facts`任务应在与主机模式匹配的所有受管主机上运行。此任务期间的故障可能导致受管主机从play中移除。
- 如果清单中明确列出了IP地址，而不是主机名，则可以将其用作主机模式。如果IP地址未列在清单中，就无法用它来指定主机，即使该IP地址会在DNS中解析到这个主机名。

示例如何使用主机模式来引用清单中包含的IP地址：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: 192.168.2.1
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [192.168.2.1]
...output omitted...
```
注意：
- 在清单中通过IP地址引用受管主机存在一个问题，那就是难以记住play或临时命令所针对的主机使用了哪个IP地址
- 如果主机没有可解析的主机名，可能必须按IP地址指定主机以进行连接
- 可以通过设置`ansible_host`主机变量，在清单中将某一别名指向特定的IP地址。例如，清单中可以有一个名为`dummy.example`的主机，然后通过创建含有以下主机变量的`host_vars/dummy.example`文件，将使用该名称的连接指向IP地址`192.168.2.1`：
    ```
    ansible_host: 192.168.2.1
    ```

#### 使用组指定主机
&#8195;&#8195;前面使用了清单主机组作为主机模式。当组名称用作主机模式时，它指定Ansible将对属于该组的成员的主机执行操作。示例如下：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: lab
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************

TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [labhost2.example.com]
...output omitted...
```
有一个名为`all`的特别组，它匹配清单中的所有受管主机：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: all
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [web.example.com]
ok: [data.example.com]
ok: [labhost1.example.com]
ok: [192.168.2.1]
ok: [test1.example.com]
ok: [192.168.2.2]
```
名为`ungrouped`的特别组，它包括清单中不属于任何其他组的所有受管主机：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: ungrouped
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [web.example.com]
ok: [data.example.com]
```
#### 使用通配符匹配多个主机
&#8195;&#8195;若要达成与`all`主机模式相同的目标，另一种方法是使用星号`*`通配符，它将匹配任意字符串。如果主机模式只是带引号的星号，则清单中的所有主机都将匹配。示例：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: '*'
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] ***********************************************
TASK [Gathering Facts] **************************************************
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [web.example.com]
ok: [data.example.com]
ok: [labhost1.example.com]
ok: [192.168.2.1]
ok: [test1.example.com]
ok: [192.168.2.2]
```
注意：
- 一些在主机模式中使用的字符对shell也有意义。通过ansible使用主机模式从命令行运行临时命令时，这可能会有问题。建议在命令行中用单引号括起使用的主机模式，防止它们被shell 
- 如果在Ansible Playbook中使用了任何特殊通配符或列表字符，必须将主机模式放在单引号里，确保能够正确解析主机模式。示例
    ```yml
    ---
    hosts: '!test1.example.com,development'
    ```

&#8195;&#8195;也可使用星号字符，以匹配包含特定子字符串的受管主机或组。示例通配符主机模式匹配以`.example.com`结尾的所有清单名称：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: '*.example.com'
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] ***********************************************
TASK [Gathering Facts] **************************************************
ok: [labhost1.example.com]
ok: [test1.example.com]
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [web.example.com]
ok: [data.example.com]
```
示例使用通配符主机模式来匹配开头为`192.168.2.`的主机或主机组的名称：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: '192.168.2.*'
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] ***********************************************
TASK [Gathering Facts] **************************************************
ok: [192.168.2.1]
ok: [192.168.2.2]
```
示例使用通配符主机模式来匹配开头为`datacenter`的主机或主机组的名称：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: 'datacenter*'
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [test1.example.com]
ok: [labhost2.example.com]
ok: [test2.example.com]
```
&#8195;&#8195;通配符主机模式匹配所有清单名称、主机和主机组。它们不区别名称是DNS名、IP地址还是组，这可能会导致一些意外的匹配。例如，根据示例清单，比较上一示例中指定`datacenter*`主机模式的结果和`data*`主机模式的结果：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: 'data*'
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [test1.example.com]
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [data.example.com]
```
#### 列表
&#8195;&#8195;可以通过逻辑列表来引用清单中的多个条目。主机模式的逗号分隔列表匹配符合任何这些主机模式的所有主机。如果提供受管主机的逗号分隔列表，则所有这些受管主机都将是目标：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: labhost1.example.com,test2.example.com,192.168.2.2
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] ***********************************************
TASK [Gathering Facts] **************************************************
ok: [labhost1.example.com]
ok: [test2.example.com]
ok: [192.168.2.2]
```
如果提供组的逗号分隔列表，则属于任何这些组的所有主机都将是目标：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: lab,datacenter1
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] ***********************************************
TASK [Gathering Facts] **************************************************
ok: [labhost1.example.com]
ok: [labhost2.example.com]
ok: [test1.example.com]
```
也可以混合使用受管主机、主机组和通配符：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: lab,data*,192.168.2.2
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [labhost2.example.com]
ok: [test1.example.com]
ok: [test2.example.com]
ok: [data.example.com]
ok: [192.168.2.2]
```
注意事项：
- 也可以用冒号`:`来取代逗号。不过，逗号是首选的分隔符，特别是将IPv6地址用作受管主机名称时

&#8195;&#8195;如果列表中的某一项以与符号`&`开头，则主机必须与该项匹配才能匹配主机模式。它的工作方式类似于逻辑`AND`。例如，根据示例清单，以下主机模式将匹配`lab`组中同时也属于`datacenter1`组的计算机：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: lab,&datacenter1
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
```
&#8195;&#8195;上述示例也可以通过主机模式`&lab,datacenter1`或`datacenter1,&lab`指定`datacenter1`组中的计算机只有在同时也属于组`lab`时才匹配。    
&#8195;&#8195;可以通过在主机模式的前面使用感叹号或惊叹号`!`，从列表中排除匹配某一模式的主机。工作方式类似于逻辑`NOT`。根据示例清单，以下示例匹配`datacenter`组中定义的所有主机，但`test2.example.com`除外：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: datacenter,!test2.example.com
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [labhost1.example.com]
ok: [test1.example.com]
ok: [labhost2.example.com]
```
&#8195;&#8195;上一示例中也可使用模式`'!test2.example.com,datacenter'`来获得同样的结果。下面示例使用匹配测试清单中的所有主机的主机模式，`datacenter1`组中的受管主机除外：
```
[student@controlnode ~]$ cat playbook.yml
---
- hosts: all,!datacenter1
...output omitted...
[student@controlnode ~]$ ansible-playbook playbook.yml
PLAY [Test Host Patterns] **************************************************
TASK [Gathering Facts] *****************************************************
ok: [web.example.com]
ok: [data.example.com]
ok: [labhost2.example.com]
ok: [test2.example.com]
ok: [192.168.2.1]
ok: [192.168.2.2]
```
## 包含和导入文件
### 管理大型Playbook
&#8195;&#8195;如果playbook很长或很复杂，可以将其分成较小的文件以便于管理。可采用模块化方式将多个playbook组合为一个主要playbook，或者将文件中的任务列表插入play。
### 包含或导入文件
包含或导入文件：
- Ansible可以使用两种操作将内容带入playbook。可以包含内容，也可以导入内容
- 包含内容是一个动态操作。在playbook运行期间，Ansible会在内容到达时处理所包含的内容
- 导入内容是一个静态操作。在运行开始之前，Ansible在最初解析playbook时预处理导入的内容

### 导入Playbook
&#8195;&#8195;`import_playbook`指令允许将包含play列表的外部文件导入playbook，可以把一个或者多个额外playbook导入到主playbook中：
- 由于导入的内容是一个完整的playbook，因此`import_playbook`功能只能在playbook的顶层使用，不能在 play内使用
- 如果导入多个playbook，则将按顺序导入并运行它们

导入两个额外playbook的主playbook的简单示例如下所示：
```yml
- name: Prepare the web server
  import_playbook: web.yml

- name: Prepare the database server
  import_playbook: db.yml
```
还可以使用导入的playbook在主playbook中交替play：
```yml
- name: Play 1
  hosts: localhost
  tasks:
    - debug:
        msg: Play 1

- name: Import Playbook
  import_playbook: play2.yml
```
上面的示例中，`Play 1`首先运行，然后运行从`play2.yml`playbook中导入的play。
### 导入和包含任务
&#8195;&#8195;可以将任务文件中的任务列表导入或包含在play中。任务文件是包含一个任务平面列表的文件，示例如下：
```
[admin@node ~]$ cat webserver_tasks.yml
- name: Installs the httpd package
  yum:
    name: httpd
    state: latest

- name: Starts the httpd service
  service:
    name: httpd
    state: started
```
#### 导入任务文件
&#8195;&#8195;可以使用`import_tasks`功能将任务文件静态导入playbook内的play中。导入任务文件时，在解析该playbook时将直接插入该文件中的任务。Playbook中的`import_tasks`的位置控制插入任务的位置以及运行多个导入的顺序。
```yml
---
- name: Install web server
  hosts: webservers
  tasks:
  - import_tasks: webserver_tasks.yml
```
&#8195;&#8195;导入任务文件时，在解析该playbook时将直接插入该文件中的任务。由于`import_tasks`在解析playbook时静态导入任务，因此对其工作方式有一些影响：
- 使用`import_tasks`功能时，导入时设置的`when`等条件语句将应用于导入的每个任务
- 无法将循环用于`import_tasks`功能
- 如果使用变量来指定要导入的文件的名称，那么将无法使用主机或组清单变量

#### 包含任务文件
也可以使用`include_tasks`功能将任务文件动态导入playbook内的play中：
```yml
---
- name: Install web server
  hosts: webservers
  tasks:
  - include_tasks: webserver_tasks.yml
```
&#8195;&#8195;在play运行并且这部分play到达之前，`include_tasks`功能不会处理playbook中的内容。Playbook内容的处理顺序会影响包含任务功能的工作方式：
- 使用`include_tasks`功能时，包含时设置的`when`等条件语句将确定任务是否包含在play中
- 如果运行`ansible-playbook --list-tasks`以列出playbook中的任务，则不会显示已包含任务文件中的任务。将显示包含任务文件的任务。相比之下，`import_tasks`功能不会列出导入任务文件的任务，而列出已导入任务文件中的各个任务。
- 不能使用`ansible-playbook --start-at-task`从已包含任务文件中的任务开始执行playbook
- 不能使用`notify`语句触发已包含任务文件中的处理程序名称。可以在包含整个任务文件的主playbook中触发处理程序，在这种情况下，已包含文件中的所有任务都将运行

#### 任务文件的用例
在这些情景中将任务组作为与playbook独立的外部文件来管理或许有所帮助：
- 如果新服务器需要全面配置，则管理员可以创建不同的任务集合，分别用于创建用户、安装软件包、配置服务、配置特权、设置对共享文件系统的访问权限、强化服务器、安装安全更新，以及安装监控代理等。每一任务集合可通过单独的自包含任务文件进行管理
- 如果服务器由开发人员、系统管理员和数据库管理员统一管理，则每个组织可以编写自己的任务文件，再由系统经理进行审核和集成
- 如果服务器要求特定的配置，它可以整合为按照某一条件来执行的一组任务。换句话说，仅在满足特定标准时才包含任务
- 如果一组服务器需要运行某一项/组任务，则它/它们可以仅在属于特定主机组的服务器上运行

#### 管理任务文件
&#8195;&#8195;可以创建专门用于任务文件的目录，并将所有任务文件保存在该目录中。然后playbook就可以从该目录包含或导入任务文件。这便能够构建复杂的playbook，同时简化其结构和组件的管理。
### 为外部play和任务定义变量
&#8195;&#8195;使用Ansible的导入和包含功能将外部文件中的play或任务合并到playbook中极大地增强了在Ansible环境中重用任务和playbook的能力。为了最大限度地提高重用可能性，这些任务和play文件应尽可能通用。变量可用于参数化play和任务元素，以扩大任务和play的应用范围。例如，以下任务文件将安装Web服务所需的软件包，然后启用并启动必要的服务：
```yml
---
  - name: Install the httpd package
    yum:
      name: httpd
      state: latest
  - name: Start the httpd service
    service:
      name: httpd
      enabled: true
      state: started
```
&#8195;&#8195;如下例所示对软件包和服务元素进行参数化，则任务文件也可用于安装和管理其他软件及其服务，而不仅仅用于Web服务：
```yml
---
  - name: Install the {{ package }} package
    yum:
      name: "{{ package }}"
      state: latest
  - name: Start the {{ service }} service
    service:
      name: "{{ service }}"
      enabled: true
      state: started
```
再将任务文件合并到一个playbook中时，定义用于执行该任务的变量，如下所示：
```yml
...output omitted...
  tasks:
    - name: Import task file and set variables
      import_tasks: task.yml
      vars:
        package: httpd
        service: httpd
```
&#8195;&#8195;Ansible使传递的变量可用于从外部文件导入的任务。可以使用相同的技术使play文件更具可重用性。将play文件合并到playbook中时，传递变量以用于执行该play，如下所示：
```yml
...output omitted...
- name: Import play file and set the variable
  import_playbook: play.yml
  vars:
    package: mariadb
```
## 练习