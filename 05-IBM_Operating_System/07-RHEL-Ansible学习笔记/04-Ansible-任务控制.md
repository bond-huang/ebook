# Ansible-任务控制
## 编写循环和条件任务
### 利用循环迭代任务
&#8195;&#8195;Ansible支持使用`loop`关键字对一组项目迭代任务。可以配置循环以利用列表中的各个项目、列表中各个文件的内容、生成的数字序列或更为复杂的结构来重复任务。
#### 简单循环
&#8195;&#8195;简单循环对一组项目迭代任务。`loop`关键字添加到任务中，将应对其迭代任务的项目列表取为值。循环变量`item`保存每个迭代过程中使用的值。以下代码片段使用两次`service`模块来确保两个网络服务处于运行状态：
```yaml
- name: Postfix is running
  service:
    name: postfix
    state: started

- name: Dovecot is running
  service:
    name: dovecot
    state: started
```
这两个任务可以重新编写为使用一个简单循环：
```yaml
- name: Postfix and Dovecot are running
  service:
    name: "{{ item }}"
    state: started
  loop:
    - postfix
    - dovecot
```
&#8195;&#8195;可以通过一个变量提供`loop`所使用的列表。下面示例中，变量`mail_services`含有需要处于运行状态的服务的列表：
```yaml
vars:
  mail_services:
    - postfix
    - dovecot

tasks:
  - name: Postfix and Dovecot are running
    service:
      name: "{{ item }}"
      state: started
    loop: "{{ mail_services }}"
```
#### 循环散列或字典列表
&#8195;&#8195;`loop`列表不需要是简单值列表。下面示例中，列表中的每个项实际上是散列或字典。示例中的每个散列或字典具有两个键，即`name`和`groups`，当前`item`循环变量中每个键的值可以分别通过`item.name`和`item.groups`变量来检索。
```yaml
- name: Users exist and are in the correct groups
  user:
    name: "{{ item.name }}"
    state: present
    groups: "{{ item.groups }}"
  loop:
    - name: ansible
      groups: wheel
    - name: huang
      groups: root
```
任务的结果是用户`ansible`存在且为组`wheel`的成员，并且用户`huang`存在且为组`root`的成员。
#### 较早样式的循环关键字
&#8195;&#8195;在Ansible 2.5之前，大多数playbook使用不同的循环语法(从Ansible2.5开始，建议使用 loop关键字编写循环)。提供了多个循环关键字，前缀为 `with_`，后跟Ansible查找插件（一项高级功能）的名称。示例如下表：

循环关键字|描述
:---:|:---
with_items|行为与简单列表的loop关键字相同，例如字符串列表或散列/字典列表。但与loop不同的是，如果为 with_items提供了列表的列表，它们将被扁平化为单级列表。循环变量item保存每次迭代过程中使用的列表项
with_file|此关键字需要控制节点文件名列表。循环变量item在每次迭代过程中保存文件列表中相应文件的内容
with_sequence|此关键字不需要列表，而是需要参数来根据数字序列生成值列表。循环变量item在每次迭代过程中保存生成的序列中的一个生成项的值

playbook中的`with_items`的示例如下所示：
```yaml
vars:
  data:
    - huang
    - ansible
    - christ
tasks:
  - name: "with_items"
    debug:
      msg: "{{ item }}"
    with_items: "{{ data }}"
```
#### 将Register变量与Loop一起使用
`register`关键字也可以捕获循环任务的输出。示例显示了循环任务中`register`变量的结构：
```yaml
---
- name: Loop Register Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Looping Echo Task
      shell: "echo This is my item: {{ item }}"
      loop:
        - one
        - two
      register: echo_results ### echo_results变量已注册

    - name: Show echo_results variable
      debug:
        var: echo_results ### echo_results 变量的内容显示在屏幕上
```
运行Playbook后输出如下(不包含星号注释)：
```
[ansible@redhat9 ~]$ ansible-playbook loop_register.yml
PLAY [Loop Register Test] ******************************************************************
TASK [Looping Echo Task] *******************************************************************
changed: [redhat8] => (item=one)
changed: [redhat8] => (item=two)
TASK [Show echo_results variable] ********************************************************************************************
ok: [redhat8] => {
    "echo_results": {   ### {字符表示echo_results变量的开头由键值对组成 ###
        "changed": true,
        "msg": "All items completed",
        "results": [  ### results键包含上一个任务的结果。[字符表示列表的开头###
            {   ###第一项的任务元数据的开头（由item键表示）。echo命令的输出可在 stdout键中找到 ###
                "ansible_facts": {
                    "discovered_interpreter_python": "/usr/libexec/platform-python"
                },
                "ansible_loop_var": "item",
                "changed": true,
                "cmd": "echo This is my item: one",
                "delta": "0:00:00.003550",
                "end": "2022-10-19 13:13:12.805972",
                "failed": false,
                "invocation": {
                    "module_args": {
                        "_raw_params": "echo This is my item: one",
                        ...output omitted...
                    }
                },
                "item": "one",
                "msg": "",
                "rc": 0,
                "start": "2022-10-19 13:13:12.802422",
                "stderr": "",
                "stderr_lines": [],
                "stdout": "This is my item: one",
                "stdout_lines": [
                    "This is my item: one"
                ]
            },
            {   ### 第二项的任务结果元数据的开头 ###
                "ansible_loop_var": "item",
                "changed": true,
                "cmd": "echo This is my item: two",
                "delta": "0:00:00.003328",
                "end": "2022-10-19 13:13:13.262803",
                "failed": false,
                "invocation": {
                    "module_args": {
                        "_raw_params": "echo This is my item: two",
                        ...output omitted...
                    }
                },
                "item": "two",
                "msg": "",
                "rc": 0,
                "start": "2022-10-19 13:13:13.259475",
                "stderr": "",
                "stderr_lines": [],
                "stdout": "This is my item: two",
                "stdout_lines": [
                    "This is my item: two"
                ]
            }
        ],  ### ]字符表示results列表的结尾 ###
        "skipped": false
    }
}
PLAY RECAP *****************************************************************************
redhat8  : ok=2  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
```
上面示例中，`results`键包含一个列表。下面示例修改了playbook，使第二个任务迭代此列表：
```
[ansible@redhat9 ~]$ cat new_loop_register.yml
---
- name: Loop Register Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Looping Echo Task
      shell: "echo This is my item: {{ item }}"
      loop:
        - one
        - two
      register: echo_results

    - name: Show stdout from the previous task.
      debug:
        meg: "STDOUT from previous task: {{ item.stdout }}"
      loop: "{{ echo_results['results'] }}"
```
运行Playbook后输出示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook new_loop_register.yml
PLAY [Loop Register Test] ******************************************************
TASK [Looping Echo Task] *******************************************************
changed: [redhat8] => (item=one)
changed: [redhat8] => (item=two)
TASK [Show stdout from the previous task.] *************************************
ok: [redhat8] => (item={...output omitted...}) => {
    "msg": "STDOUT from previous task: This is my item: one"
}
ok: [redhat8] => (item={...output omitted...}) => {
    "msg": "STDOUT from previous task: This is my item: two"
}
PLAY RECAP *********************************************************************
redhat8  : ok=2  changed=1  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0
```
### 有条件地运行任务
&#8195;&#8195;Ansible可使用`conditionals`在符合特定条件时执行任务或play。管理员可利用条件来区分不同的受管主机，并根据它们所符合的条件来分配功能角色。Playbook变量、注册的变量和Ansible事实都可通过条件来进行测试，可以使用比较字符串、数字数据和布尔值的运算符。以下场景说明了在 Ansible 中使用条件的情况：
- 可以在变量中定义硬限制（如min_memory）并将它与受管主机上的可用内存进行比较
- Ansible可以捕获并评估命令的输出，以确定某一任务在执行进一步操作前是否已经完成。例如，如果某一程序失败，则将跳过批处理
- 可以利用Ansible事实来确定受管主机网络配置，并且决定要发送的模板文件（如，网络绑定或中继）
- 可以评估CPU的数量，来确定如何正确调节某一 Web服务器
- 将注册的变量与预定义的变量进行比较，以确定服务是否已更改。例如，测试服务配置文件的MD5校验和以查看服务是否已更改

#### 条件任务语法
&#8195;&#8195;`when`语句用于有条件地运行任务，取要测试的条件作为值，如果条件满足，则运行任务，如何条件不满足，则跳过任务。可以测试的一个最简单条件是某一布尔变量是`true`还是`false`。示例任务仅在`run_my_task`为`true`时运行：
```yaml
---
- name: Simple Boolean Task Demo
  hosts: all
  vars:
    run_my_task: true

  tasks:
    - name: httpd package is installed
      yum:
        name: httpd
      when: run_my_task
```
&#8195;&#8195;下面示例测试`my_service`变量是否具有值，若有值，则将`my_service`的值用作要安装的软件包的名称，如果未定义`my_service`变量，则跳过任务且不显示错误：
```yaml
---
- name: Test Variable is Defined Demo
  hosts: all
  vars:
    my_service: httpd

  tasks:
    - name: "{{ my_service }} package is installed"
      yum:
        name: "{{ my_service }}"
      when: my_service is defined
```
下表是在处理条件时可使用的一些运算：

操作|示例
:---:|:---
等于(值为字符串)|ansible_machine == "x86_64"
等于(值为数字)|max_memory == 512
小于|min_memory < 128
大于|min_memory > 256
小于等于|min_memory <= 256
大于等于|min_memory >= 512
不等于|min_memory != 512
变量存在|min_memory is defined
变量不存在|min_memory is not defined
布尔变量是true。1、True或yes的求值为true|memory_available
布尔变量是false。0、False或no的求值为false|not memory_available
第一个变量的值存在，作为第二个变量的列表中的值|ansible_distribution in supported_distros

&#8195;&#8195;上表中的最后一个条目不好理解，下面示例演示了它的工作原理。在示例中，`ansible_distribution`变量是在`Gathering Facts`任务期间确定的事实，用于标识托管主机的操作系统分发。变量`supported_distros`由 playbook作者创建，包含该playbook支持的操作系统分发列表。如果`ansible_distribution`的值在 `supported_distros`列表中，则条件通过且任务运行：
```yaml
---
- name: Demonstrate the "in" keyword
  hosts: all
  gather_facts: yes
  vars:
    supported_distros:
      - RedHat
      - Fedora
  tasks:
    - name: Install httpd using yum, where supported
      yum:
        name: http
        state: present
      when: ansible_distribution in supported_distros
```
注意事项：
- 注意`when`语句的缩进。由于`when`语句不是模块变量，它必须通过缩进到任务的最高级别，放置在模块的外面。

#### 测试多个条件
&#8195;&#8195;一个`when`语句可用于评估多个条件，可以使用`and`或`or`关键字组合条件，并使用括号分组条件。如果任一条件为真时满足条件语句，则应当使用 or 语句。例如，如果计算机上运行的是红帽企业 Linux 或 Fedora，则下述条件得到满足：
```yaml
when: ansible_distribution == "RedHat" or ansible_distribution == "Fedora"
```
&#8195;&#8195;使用`and`运算时，两个条件都必须为真，才能满足整个条件语句。例如，如果远程主机是红帽企业Linux7.5主机，并且安装的内核是指定版本，则将满足以下条件：
```yaml
when: ansible_distribution_version == "7.5" and ansible_kernel == "3.10.0-327.el7.x86_64"
```
&#8195;&#8195;`when`关键字还支持使用列表来描述条件列表。向`when`关键字提供列表时，将使用`and`运算组合所有条件。示例使用`and`运算符组合多个条件语句的另一方式(这种格式提高了可读性)：
```yaml
when:
  - ansible_distribution_version == "7.5"
  - ansible_kernel == "3.10.0-327.el7.x86_64"
```
&#8195;&#8195;通过使用括号分组条件，可以表达更复杂的条件语句。例如，如果计算机上运行的是红帽企业Linux 7或Fedora 28，则下述条件语句得到满足。此示例使用大于字符(>)，这样长条件就可以在playbook中分成多行，以便于阅读：
```yaml
when: >
    ( ansible_distribution == "RedHat" and
      ansible_distribution_major_version == "7" )
    or
    ( ansible_distribution == "Fedora" and
    ansible_distribution_major_version == "28" )
```
### 组合循环和有条件任务
&#8195;&#8195;循环和条件可以组合使用。下面示例中，`yum`模块将安装`mariadb-server`软件包，只要`/`上挂载的文件系统具有超过 `300MB`的可用空间。`ansible_mounts`事实是一组字典，各自代表一个已挂载文件系统的相关事实。循环迭代列表中的每一字典，只有找到了代表两个条件都为真的已挂载文件系统的字典时，条件语句才得到满足：
```yaml
- name: install mariadb-server if enough space on root
  yum:
    name: mariadb-server
    state: latest
  loop: "{{ ansible_mounts }}"
  when: item.mount == "/" and item.size_available > 300000000
```
&#8195;&#8195;下面是组合使用条件和注册变量的另一个示例。下方标注的playbook只有在`postfix`服务处于运行状态时才会重新启动`httpd`服务：
```yaml
---
- name: Restart HTTPD if Postfix is Running
  hosts: all
  tasks:
    - name: Get Postfix server status
      command: /usr/bin/systemctl is-active postfix ### Postfix是否在运行？
      ignore_errors: yes ### 如果它不在运行并且命令失败，则不停止处理
      register: result ### 将模块的结果信息保存在名为result的变量中

    - name: Restart Apache HTTPD based on Postfix status
      service:
        name: httpd
        state: restarted
      when: result.rc == 0 ### 评估Postfix任务的输出。如果systemctl命令的退出代码为0，则Postfix激活并且此任务重新启动httpd服务
```
## 实施处理程序
### Ansible处理程序
&#8195;&#8195;`Ansible`模块设计为具有幂等性。这表示，在正确编写的playbook中，playbook及其任务可以运行多次而不会改变受管主机，除非需要进行更改使受管主机进入所需的状态。但有时候，在任务确实更改系统时，可能需要运行进一步的任务。例如，更改服务配置文件时可能要求重新加载该服务以便使更改的配置生效。    
&#8195;&#8195;处理程序是响应由其他任务触发的通知的任务。仅当任务在受管主机上更改了某些内容时，任务才通知其处理程序。每个处理程序在play的任务块后都是由其名称触发的：
- 如果没有任务通过名称通知处理程序，处理程序就不会运行
- 如果一个或多个任务通知处理程序，处理程序就会在play中的所有其他任务完成后运行一次。因为处理程序就是任务，所以管理员可以在处理程序中使用他们将用于任何其他任务的模块
- 通常而言，处理程序被用于重新引导主机和重新启动服务
- 使用唯一名称来命名您的处理程序。使用相同名称定义多个处理程序时，只会运行相同名称的最后一个处理程序

&#8195;&#8195;处理程序可视为非活动任务，只有在使用`notify`语句显式调用时才会被触发。在下列代码片段中，只有配置文件更新并且通知了该任务时，`restart apache`处理程序才会重新启动`Apache`服务器：
```yaml
tasks:
  - name: copy demo.example.conf configuration template ### 通知处理程序的任务
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify: ### notify语句指出该任务需要触发一个处理程序
      - restart apache ### 要运行的处理程序的名称

handlers: ### handlers关键字表示处理程序任务列表的开头
  - name: restart apache ### 被任务调用的处理程序的名称
    service: ### 用于该处理程序的模块
      name: httpd
      state: restarted
```
&#8195;&#8195;示例中，`restart apache`处理程序只有在`template`任务通知已发生更改时才会触发。一个任务可以在其`notify`部分中调用多个处理程序。Ansible将`notify`语句视为数组，并且迭代处理程序名称：
```yaml
tasks:
  - name: copy demo.example.conf configuration template
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart mysql
      - restart apache

handlers:
  - name: restart mysql
    service:
      name: mariadb
      state: restarted

  - name: restart apache
    service:
      name: httpd
      state: restarted
```
### 使用处理程序的好处
使用处理程序时需要牢记几个重要事项：
- 处理程序始终按照`play`的`handlers`部分指定的顺序运行。它们不按在任务中由`notify`语句列出的顺序运行，或按任务通知它们的顺序运行
- 处理程序通常在相关`play`中的所有其他任务完成后运行。playbook的`tasks`部分中某一任务调用的处理程序，将等到`tasks`下的所有任务都已处理后才会运行(极少例外)
- 处理程序名称存在于各`play`命名空间中。如果两个处理程序被错误地给予相同的名称，则仅会运行一个
- 即使有多个任务通知处理程序，该处理程序依然仅运行一次。如果没有任务通知处理程序，它就不会运行
- 如果包含`notify`语句的任务没有报告`changed`结果（例如，软件包已安装并且任务报告ok），则处理程序不会获得通知。处理程序将被跳过，直到有其他任务通知它。只有相关任务报告了`changed`状态，Ansible才会通知处理程序

注意事项：
- 处理程序用于在任务对受管主机进行更改时执行额外操作。它们不应用作正常任务的替代

## 处理任务失败
### 管理Play中的任务错误
&#8195;&#8195;Ansible评估各任务的返回代码，从而确定任务是成功还是失败。通常而言，当任务失败时，Ansible将立即在该主机上中止play的其余部分并且跳过所有后续任务。但有些时候，可能希望即使在任务失败时也继续执行play。例如，预期特定任务有可能会失败，并且希望通过有条件地运行某项其他任务来恢复。有多种Ansible功能可用于管理任务错误。
#### 忽略任务失败
&#8195;&#8195;默认情况下，任务失败时`play`会中止。不过，可以通过忽略失败的任务来覆盖此行为。可以在任务中使用`ignore_errors`关键字来实现此目的。例如，如果`notapkg`软件包不存在，则`yum`模块将失败，但若将 `ignore_errors`设为 yes，则执行将继续：
```yaml
- name: Latest version of notapkg is installed
  yum:
    name: notapkg
    state: latest
  ignore_errors: yes
```
#### 任务失败后强制执行处理程序
&#8195;&#8195;通常如果任务失败并且`play`在该主机上中止，则收到`play`中早前任务通知的处理程序将不会运行。如果在`play`中设置`force_handlers: yes`关键字，则即使`play`因为后续任务失败而中止也会调用被通知的处理程序。示例如下：
```yaml
---
- hosts: all
  force_handlers: yes
  tasks:
    - name: a task which always notifies its handler
      command: /bin/true
      notify: restart the database

    - name: a task which fails because the package doesn't exist
      yum:
        name: notapkg
        state: latest

  handlers:
    - name: restart the database
      service:
        name: mariadb
        state: restarted
```
注意事项：
- 处理程序会在任务报告`changed`结果时获得通知，而在任务报告`ok`或`failed`结果时不会获得通知

#### 指定任务失败条件
&#8195;&#8195;可以在任务中使用`failed_when`关键字来指定表示任务已失败的条件。这通常与命令模块搭配使用，这些模块可能成功执行了某一命令，但命令的输出可能指示了失败。例如，可以运行输出错误消息的脚本，并使用该消息定义任务的失败状态。示例如下：
```yaml
tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
```
`fail`模块也可用于强制任务失败。上面的场景也可以编写为两个任务：
```yaml
tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: yes

  - name: Report script failure
    fail:
      msg: "The password is missing in the output"
    when: "'Password missing' in command_result.stdout"
```
&#8195;&#8195;可以使用`fail`模块为任务提供明确的失败消息。此方法还支持延迟失败，允许运行中间任务以完成或回滚其他更改。
#### 指定任务何时报告“Changed”结果
&#8195;&#8195;当任务对托管主机进行了更改时，会报告`changed`状态并通知处理程序。如果任务不需要进行更改，则会报告`ok`并且不通知处理程序。`changed_when`关键字可用于控制任务在何时报告它已进行了更改。示例如下：
```yaml
  - name: get Kerberos credentials as "admin"
    shell: echo "{{ krb_admin_pass }}" | kinit -f admin
    changed_when: false
```
&#8195;&#8195;示例中的`shell`模块将用于获取供后续任务使用的`Kerberos`凭据。它通常会在运行时始终报告`changed`。为抑制这种更改，应设置`changed_when: false`，以便它仅报告`ok`或`failed`。下面示例使用`shell`模块，根据通过已注册变量收集的模块的输出来报告`changed`：
```yaml
tasks:
  - shell:
      cmd: /usr/local/bin/upgrade-database
    register: command_result
    changed_when: "'Success' in command_result.stdout"
    notify:
      - restart_database

handlers:
  - name: restart_database
     service:
       name: mariadb
       state: restarted
```
#### Ansible块和错误处理
&#8195;&#8195;在playbook中，块是对任务进行逻辑分组的子句，可用于控制任务的执行方式。例如，任务块可以含有`when`关键字，以将某一条件应用到多个任务：
```yaml
- name: block example
  hosts: all
  tasks:
    - name: installing and configuring Yum versionlock plugin
      block:
      - name: package needed by yum
        yum:
          name: yum-plugin-versionlock
          state: present
      - name: lock version of tzdata
        lineinfile:
          dest: /etc/yum/pluginconf.d/versionlock.list
          line: tzdata-2016j-1
          state: present
      when: ansible_distribution == "RedHat"
```
&#8195;&#8195;通过块，也可结合`rescue`和`always`语句来处理错误。如果块中的任何任务失败，则执行其`rescue`块中的任务来进行恢复。在`block`子句中的任务以及`rescue`子句中的任务（如果出现故障）运行之后，`always`子句中的任务运行。总结：
- block：定义要运行的主要任务
- rescue：定义要在`block`子句中定义的任务失败时运行的任务
- always：定义始终都独立运行的任务，不论`block`和`rescue`子句中定义的任务是成功还是失败

&#8195;&#8195;示例演示了如何在playbook中实施块。即使`block`子句中定义的任务失败，`rescue`和`always`子句中定义的任务也会执行(`block`中的`when`条件也会应用到其`rescue`和`always`子句)：
```yaml
tasks:
    - name: Upgrade DB
      block:
        - name: upgrade the database
          shell:
            cmd: /usr/local/lib/upgrade-database
      rescue:
        - name: revert the database upgrade
          shell:
            cmd: /usr/local/lib/revert-database
      always:
        - name: always restart the database
          service:
            name: mariadb
            state: restarted
```
## 练习