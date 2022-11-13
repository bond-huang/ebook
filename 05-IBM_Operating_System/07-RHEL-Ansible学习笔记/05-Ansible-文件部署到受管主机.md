# Ansible-将文件部署到受管主机
## 修改文件并将其复制到主机
### 描述文件模块
&#8195;&#8195;`Files`模块库包含的模块允许用户完成与Linux文件管理相关的大多数任务，如创建、复制、编辑和修改文件的权限和其他属性。下表是常用文件管理模块的列表：

模块名称|模块说明
:---:|:---
blockinfile|插入、更新或删除由可自定义标记线包围的多行文本块
copy|将文件从本地或远程计算机复制到受管主机上的某个位置。类似于file模块，copy模块还可以设置文件属性，包括SELinux上下文。
fetch|此模块的作用和copy模块类似，但以相反方式工作。此模块用于从远程计算机获取文件到控制节点，并将它们存储在按主机名组织的文件树中
file|设置权限、所有权、SELinux上下文以及常规文件、符号链接、硬链接和目录的时间戳等属性。此模块还可以创建或删除常规文件、符号链接、硬链接和目录。其他多个与文件相关的模块支持与file模块相同的属性设置选项，包括copy模块
lineinfile|确保特定行位于某个文件中，或使用反向引用正则表达式来替换现有行。此模块主要在您想要更改文件中的某一行时使用
stat|检索文件的状态信息，类似于Linux stat命令
synchronize|围绕rsync命令的一个打包程序，可加快和简化常见任务。synchronize模块无法提供对rsync命令的完整功能的访问权限，但确实使最常见的调用更容易实施。根据您的用例，您可能仍需要通过run command模块直接调用rsync命令

### Files模块的自动化示例
&#8195;&#8195;在受管主机上创建、复制、编辑和删除文件是您可以使用`Files`模块库中的模块实施的常见任务。以下示例显示了可以使用这些模块自动执行常见文件管理任务的方式。
#### 确保受管主机上存在文件
&#8195;&#8195;使用`file`模块处理受管主机上的文件的工作方式与`touch`命令类似，如果不存在则创建一个空文件，如果存在，则更新其修改时间。在本例中，除了处理文件之外，Ansible还确保将文件的拥有用户、组和权限设置为特定值。
```yml
---
- name: Touch a file Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Touch a file and set permissions
      file:
        path: /home/ansible/filetest
        owner: ansible
        group: ansible
        mode: 0755
        state: touch
```
运行Play示例：
```
[ansible@redhat9 ~]$ ansible-playbook file1.yml
PLAY [Touch a file Test] *****************************************************************************
TASK [Touch a file and set permissions] *****************************************************************************
changed: [redhat8]
PLAY RECAP ******************************************************************
redhat8 : ok=1 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```
在redhat8上查看文件：
```
[ansible@redhat8 ~]$ ls -l filetest
-rwxr-xr-x. 1 ansible ansible 0 Nov  8 15:33 filetest
```
#### 修改文件属性
&#8195;&#8195;可以使用`file`模块，确保新的或现有的文件具有正确的权限和`SELinux`类型。例如以下文件保留了相对于用户主目录的默认`SELinux`上下文，这不是所需的上下文：
```
[ansible@redhat8 ~]$ ls -Z filetest1
unconfined_u:object_r:user_home_t:s0 filetest1
```
&#8195;&#8195;以下任务确保了`filetest1`文件的`SELinux`上下文类型属性是所需的`samba_share_t`类型。与Linux中的`chcon`命令类似：
```yml
---
- name: Change a file Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: SELinux type is set to samba_share_t
      file:
        path: /home/ansible/filetest1
        setype: samba_share_t
```
运行后查看文件：
```
[ansible@redhat8 ~]$ ls -Z filetest1
unconfined_u:object_r:samba_share_t:s0 filetest1
```
注意：
- 文件属性参数在多个文件管理模块中可用。运行`ansible-doc file`和`ansible-doc copy`命令以获取其他信息。

#### 使SELinux文件上下文更改具有持久性
&#8195;&#8195;设置文件上下文时，`file`模块的行为与`chcon`类似。运行`restorecon`，可能会意外地撤消使用该模块所做的更改。使用`file`设置上下文后，可以使用`System`模块集合中的`sefcontext`来更新 `SELinux`策略，如`semanage fcontext`：
```yml
---
- name: SELinux type Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: SELinux type is persistently set to samba_share_t
      sefcontext:
        target: /home/ansible/filetest2
        setype: samba_share_t
        state: present
```
注意：
- 使用`sefcontext`模块需要在控制和受控节点上安装`libselinux-python`和`policycoreutils-python`
- `sefcontext`模块更新`SELinux`策略中目标的默认上下文，但不更改现有文件的上下文

#### 在受管主机上复制和编辑文件
&#8195;&#8195;`copy`模块用于将位于控制节点上的Ansible工作目录中的文件复制到选定的受管主机。默认情况下，此模块假定设置了`force: yes`。这会强制该模块覆盖远程文件（如果存在但包含与正在复制的文件不同的内容）。如果设置`force: no`，则它仅会将该文件复制到受管主机（如果该文件尚不存在）
```yml
---
- name: Copy Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Copy a file to managed hosts
      copy:
        src: copytest
        dest: /home/ansible/copytest
```
源文件查看示例：
```
[ansible@redhat9 ~]$ ls -l copytest
-rw-r--r--. 1 ansible ansible 9 Nov  9 23:18 copytest
```
运行后目标文件查看示例：
```
[ansible@redhat8 ~]$ ls -l copytest
-rw-r-----. 1 root root 9 Nov  9 15:19 copytest
```
&#8195;&#8195;使用`fetch`模块从受管主机检索文件。这可用于在将参考系统分发给其他受管主机之前从参考系统中检索诸如SSH公钥之类的文件：
```yml
---
- name: fetch Test
  gather_facts: no
  hosts: redhat8
  vars:
    user: ansible
  tasks:
    - name: Retrieve SSH key from reference host
      fetch:
        src: "/home/{{ user }}/.ssh/id_rsa.pub"
        dest: "files/keys/{{ user }}.pub"
```
要确保现有文件中存在特定的单行文本，使用`lineinfile`模块：
```yml
---
- name: lineinfile Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Add a line of text to a file
      lineinfile:
        path: /home/ansible/copytest
        line: 'Add this line to the file'
        state: present
```
运行后在目标系统查看文件示例：
```
[root@redhat8 ansible]# cat copytest
copytest
Add this line to the file
```
要将文本块添加到现有文件，使用`blockinfile`模块：
```yml
---
- name: blockinfile Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Add additional lines to a file
      blockinfile:
        path: /home/ansible/copytest
        block: |
          First line in the additional block of text
          Second line in the additional block of text
        state: present
```
运行后在目标系统查看文件示例：
```
[root@redhat8 ansible]# cat copytest
copytest
Add this line to the file
# BEGIN ANSIBLE MANAGED BLOCK
First line in the additional block of text
Second line in the additional block of text
# END ANSIBLE MANAGED BLOCK
```
注意，使用`blockinfile`模块时，注释块标记插入到块的开头和结尾，以确保幂等性：
```
# BEGIN ANSIBLE MANAGED BLOCK
First line in the additional block of text
Second line in the additional block of text
# END ANSIBLE MANAGED BLOCK
```
可以使用该模块的`marker`参数，帮助确保将正确的注释字符或文本用于相关文件。
#### 从受管主机中删除文件
&#8195;&#8195;从受管主机中删除文件的基本示例是使用`file`模块和`state: absent`参数。`state`参数对于许多模块是可选的。出于多个原因，应始终明确意图，即是需要`state: present`还是`state: absent`。一些模块也支持其他选项。默认值可能会在某个时候发生变化，但也许最重要的是可以更轻松地根据您的任务了解系统应处于的状态：
```yml
---
- name: Delete file Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Make sure a file does not exist on managed hosts
      file:
        dest: /home/ansible/filetest2
        state: absent
```
运行后在目标系统查看文件示例：
```
[ansible@redhat8 ~]$ ls -l filetest2
-rw-rw----. 1 ansible ansible 0 Nov  9 14:39 filetest2
[ansible@redhat8 ~]$ ls -l filetest2
ls: cannot access 'filetest2': No such file or directory
```
#### 检索受管主机上的文件状态
&#8195;&#8195;`stat`模块检索文件的事实，类似于Linux的`stat`命令。参数提供检索文件属性、确定文件校验和等功能。`stat`模块返回一个包含文件状态数据的值的散列字典，允许使用单独的变量引用各条信息。     
&#8195;&#8195;以下示例注册`stat`模块的结果，然后显示它检查的文件的`MD5`校验和。（也可使用SHA256算法；这里使用MD5以提高易读性）：
```yml
---
- name: Stat Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: Verify the checksum of a file
      stat:
        path: /home/ansible/copytest
        checksum_algorithm: md5
      register: result
    - debug:
        msg: "The checksum of the file is {{ result.stat.checksum }}"
```
运行后示例如下：
```
[ansible@redhat9 ~]$ ansible-playbook stat.yml
PLAY [Stat Test] ************************************************************
TASK [Verify the checksum of a file] *****************************************************************************
ok: [redhat8]
TASK [debug] ****************************************************************
ok: [redhat8] => {
    "msg": "The checksum of the file is cd9c557faefc92f92d2806227fb02982"
}
PLAY RECAP ******************************************************************
redhat8 : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```
&#8195;&#8195;有关`stat`模块返回的值的信息由`ansible-doc`记录，或者可以注册一个变量并显示其内容以查看可用内容，示例如下：
```yml
- name: Examine all stat output of /etc/passwd
  hosts: localhost
  tasks:
    - name: stat /etc/passwd
      stat:
        path: /etc/passwd
      register: results
    - name: Display stat results
      debug:
        var: results
```
#### 同步控制节点和受管主机之间的文件
&#8195;&#8195;`synchronize`模块是一个围绕`rsync`工具的打包程序，它简化了`playbook`中的常见文件管理任务。`rsync`工具必须同时安装在本地和远程主机上。默认情况下，在使用`synchronize`模块时，`本地主机`是同步任务所源自的主机（通常是控制节点），而`目标主机`是`synchronize`连接到的主机。以下示例将位于 Ansible工作目录中的文件同步到受管主机：
```yml
---
- name: Synchronizet Test
  gather_facts: no
  hosts: redhat8
  tasks:
    - name: synchronize local file to remote files
      synchronize:
        src: fetch.yml
        dest: /home/ansible/fetch.yml
```
&#8195;&#8195;有很多种方法可以使用`synchronize`模块及其许多参数，包括同步目录。运行`ansible-doc synchronize`命令查看其他参数和playbook示例。
## 使用Jinja2模板部署自定义文件
### Jinja2简介
&#8195;&#8195;Ansible将Jinja2模板系统用于模板文件。Ansible还使用Jinja2语法来引用playbook中的变量，在直接学习中已经接触到了。
#### 使用分隔符
变量和逻辑表达式置于标记或分隔符之间。例如：
- Jinja2模板将`{% EXPR %}`用于表达式或逻辑（如循环）
- `{{ EXPR }}`则用于向最终用户输出表达式或变量的结果，呈现时将被替换为一个或多个值，对最终用户可见
- 使用`{# COMMENT #}`语法括起不应出现在最终文件中的注释

&#8195;&#8195;下面示例中，第一行中含有不会包含于最终文件中的注释。第二行中引用的变量被替换为所引用的系统事实的值：
```
{# /etc/hosts line #}
{{ ansible_facts['default_ipv4']['address'] }}  {{ ansible_facts['hostname'] }}
```
### 构建Jinja2模板
Jinja2模板由多个元素组成：数据、变量和表达式：
- 在呈现Jinja2模板时，这些变量和表达式被替换为对应的值
- 模板中使用的变量可以在playbook的`vars`部分中指定
- 可以将受管主机的事实用作模板中的变量。`ansible system_hostname -i inventory_file -m setup`命令来获取与受管主机相关的事实

&#8195;&#8195;使用变量及Ansible从受管主机检索的事实创建`/etc/ssh/sshd_config`的模板。当执行相关的playbook时，任何事实都将被替换为所配置的受管主机中对应的值：
```
# {{ ansible_managed }}
# DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE AS THEY WILL BE LOST

Port {{ ssh_port }}
ListenAddress {{ ansible_facts['default_ipv4']['address'] }}

HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

SyslogFacility AUTHPRIV

PermitRootLogin {{ root_allowed }}
AllowGroups {{ groups_allowed }}

AuthorizedKeysFile /etc/.rht_authorized_keys .ssh/authorized_keys

PasswordAuthentication {{ passwords_allowed }}

ChallengeResponseAuthentication no

GSSAPIAuthentication yes
GSSAPICleanupCredentials no

UsePAM yes

X11Forwarding yes
UsePrivilegeSeparation sandbox

AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
AcceptEnv XMODIFIERS

Subsystem sftp	/usr/libexec/openssh/sftp-server
```
### 部署Jinja2模板
&#8195;&#8195;创建了适用于配置文件的Jinja2模板后，可以通过`template`模块部署到受管主机上，该模块支持将控制节点中的本地文件转移到受管主机，`template`模块使用语法：
- 与`src`键关联的值指定来源Jinja2模板
- 与`dest`键关联的值指定要在目标主机上创建的文件

示例如下：
```yml
tasks:
  - name: template render
    template:
      src: /tmp/j2-template.j2
      dest: /tmp/dest-config-file.txt
```
注意：
- `template`模块还允许指定已部署文件的所有者（拥有该文件的用户）、组、权限和SELinux上下文，像`file`模块一样。
- 它也可以取用`validate`选项运行任意命令（如`visudo -c`），在将文件复制到位之前检查该文件的语法是否正确
- 更多详细信息，参阅`ansible-doc`模板

### 管理模板文件
&#8195;&#8195;为避免系统管理员修改Ansible部署的文件，最好在模板顶部包含注释，以指示不应手动编辑该文件。可使用`ansible_managed`指令中设置的`“Ansible managed”`字符串来执行此操作。这不是正常变量，但可以在模板中用作一个变量。`ansible_managed`指令在`ansible.cfg`文件中设置：
```ini
ansible_managed = Ansible managed
```
将`ansible_managed`字符串包含在Jinja2模板内，使用下列语法：
```
{{ ansible_managed }}
```
### 控制结构
&#8195;&#8195;可以在模板文件中使用Jinja2控制结构，以减少重复键入，为play中的每个主机动态输入条目，或者有条件地将文本插入到文件中。注意：
- 可以在Ansible模板中使用Jinja2循环和条件，但不能在Ansible Playbook中使用

#### 使用循环
&#8195;&#8195;Jinja2种使用`for`语句来提供循环功能。在下面示例中，`user`变量替换为`users`变量中包含的所有值，一行一个值：
```
{% for user in users %}
      {{ user }}
{% endfor %}
```
&#8195;&#8195;下面示例中的模板使用`for`语句逐一运行`users`变量中的所有值，将`myuser`替换为各个值，但值为`root`时除外。示例中`loop.index`变量扩展至循环当前所处的索引号。它在循环第一次执行时值为`1`，每一次迭代递增`1`：
```
{# for statement #}
{% for myuser in users if not myuser == "root" %}
User number {{ loop.index }} - {{ myuser }}
{% endfor %}
```
&#8195;&#8195;下面示例此模板也使用了`for`语句，并且假定使用的清单文件中已定义了`myhosts`变量。此变量将包含要管理的主机的列表。示例中，文件中将列出清单中`myhosts`组内的所有主机。
```
{% for myhost in groups['myhosts'] %}
{{ myhost }}
{% endfor %}
```
可以使用该模板从主机事实动态生成`/etc/hosts`文件。假设有以下playbook：
```yml
- name: /etc/hosts is up to date
  hosts: all
  gather_facts: yes
  tasks:
    - name: Deploy /etc/hosts
      template:
        src: templates/hosts.j2
        dest: /etc/hosts
```
&#8195;&#8195;下述三行`templates/hosts.j2`模板从`all`组中的所有主机构造文件。（由于变量名称的长度，模板的中间行非常长）它迭代组中的每个主机以获得`/etc/hosts`文件的三个事实：
```
{% for host in groups['all'] %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['fqdn'] }} {{ hostvars[host]['ansible_facts']['hostname'] }}
{% endfor %}
```
#### 使用条件句
&#8195;&#8195;Jinja2使用`if`语句来提供条件控制。如果满足某些条件，这允许在已部署的文件中放置一行。
下面示例中，仅当`finished`变量的值为`True`时，才可将`result`变量的值放入已部署的文件：
```
{% if finished %}
{{ result }}
{% endif %}
```
### 变量过滤器
&#8195;&#8195;Jinja2提供了过滤器，更改模板表达式的输出格式（例如，输出到JSON）。有适用于YAML和 JSON等语言的过滤器。`to_json`过滤器使用JSON格式化表达式输出，`to_yaml`过滤器则使用YAML格式化表达式输出：
```
{{ output | to_json }}
{{ output | to_yaml }}
```
&#8195;&#8195;还有其它过滤器，`to_nice_json`和`to_nice_yaml`等过滤器将表达式输出格式化为JSON或YAML等人类可读格式。示例如下：
```
{{ output | to_nice_json }}
{{ output | to_nice_yaml }}
```
&#8195;&#8195;`from_json`和`from_yaml`过滤器相应要求JSON或YAML格式的字符串，并对它们进行解析。使用示例如下：
```
{{ output | from_json }}
{{ output | from_yaml }}
```
### 变量测试
&#8195;&#8195;在Ansible Playbook中与`when`子句一同使用的表达式是Jinja2表达式。用于测试返回值的内置Ansible测试包括`failed`、`changed`、`succeeded`和`skipped`。如何在条件表达式内使用测试示例：
```
tasks:
...output omitted...
  - debug: msg="the execution was aborted"
    when: returnvalue is failed
```
## 练习