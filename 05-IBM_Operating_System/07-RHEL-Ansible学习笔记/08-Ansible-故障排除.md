# Ansible-故障排除
## 对Playbook进行故障排除
### Ansible日志文件
&#8195;&#8195;默认情况下，Ansible配置为不将其输出记录到任何日志文件。它提供了一个内置日志基础架构，可以通过`ansible.cfg`配置文件的`default`部分中的`log_path`参数进行配置，或者通过`$ANSIBLE_LOG_PATH`环境变量来配置：
- 如果进行了其中任一或全部配置，Ansible会把来自`ansible`和`ansible-playbook`命令的输出存储到通过`ansible.cfg`配置文件或`$ANSIBLE_LOG_PATH`环境变量配置的日志文件中
- 如果将Ansible配置为将日志文件写入`/var/log`，则红帽建议配置`logrotate`来管理Ansible日志文件

### 调试模块
&#8195;&#8195;通过`debug`模块可以了解play中发生的情况。此模块可以显示play中某个点上某个变量的值。在对使用变量互相通信的任务（例如，将一项任务的输出用作后续任务的输入）进行调试时，此功能可以发挥关键作用。    
&#8195;&#8195;示例使用`debug`任务中的`msg`和`var`设置。第一个示例显示`ansible_facts['memfree_mb']`事实的运行时值，作为`ansible-playbook`输出中显示的消息的一部分：
```yml
- name: Display free memory
  debug:
    msg: "Free memory for this system is {{ ansible_facts['memfree_mb'] }}"
```
第二个示例显示`output`变量的值：
```yml
- name: Display the "output" variable
  debug:
    var: output
    verbosity: 2
```
### 管理错误
&#8195;&#8195;Playbook运行期间可能会发生多种问题，它们主要与Playbook或它使用的任何模板的语法相关，或者源自与受管主机的连接问题（例如，清单文件中受管主机的主机名称存在错误）。这些错误在执行时由`ansible-playbook`命令发出。     
&#8195;&#8195;`--syntax-check`选项可检查Playbook的YAML语法。在使用Playbook之前，或者当遇到了相关问题时，最好对其运行语法检查：
```
[student@demo ~]$ ansible-playbook play.yml --syntax-check
```
&#8195;&#8195;也可以使用`--step`选项来逐步调试Playbook，一次一个任务。`ansible-playbook --step`命令以交互方式提示确认希望运行的每个任务：
```
[student@demo ~]$ ansible-playbook play.yml --step
```
&#8195;&#8195;`--start-at-task`选项允许从特定的任务开始执行Playbook。它取要作为开始的任务名称作为参数，示例如下：
```
[student@demo ~]$ ansible-playbook play.yml --start-at-task="start httpd service"
```
### 调试
&#8195;&#8195;以`ansible-playbook`命令运行Playbook所提供的输出作为起点，是对Ansible受管主机相关问题进行故障排除的良好开端。参考Playbook执行的以下输出：
```
PLAY [Service Deployment] ***************************************************
...output omitted...
TASK: [Install a service] ***************************************************
ok: [demoservera]
ok: [demoserverb]

PLAY RECAP ******************************************************************
demoservera                  : ok=2    changed=0    unreachable=0    failed=0
demoserverb                  : ok=2    changed=0    unreachable=0    failed=0
```
输出说明：
- 在上面的输出中，显示了一个`PLAY`标头（带有要执行的play的名称），后跟一个或多个`TASK`标头。这些标头各自代表其在Playbook中相关的任务，它会在属于Playbook中`hosts`参数包含的组的所有受管主机上执行
- 当每一受管主机执行各个play的任务时，受管主机的名称显示在对应的`TASK`标头下，同时显示有该受管主机上的任务状态。任务状态可以显示为`ok`、`fatal`、`changed`或`skipping`
- 在各个play的输出的底部，`PLAY RECAP`部分显示对每一受管主机执行的任务的数量

&#8195;&#8195;可以通过添加一个或多个`-v`选项来提高`ansible-playbook`输出的详细程度。`ansible-playbook -v`命令提供了额外的调试信息，总共有四个级别：

选项|描述
:---:|:---
-v|显示输出数据
-vv|显示输出和输入数据
-vvv|包含关于和受管主机连接的信息
-vvvv|包括其他信息，如每一远程主机上执行的脚本，以及执行各个脚本的用户

### Playbook管理的推荐做法
&#8195;&#8195;在开发这些Playbook时，务必要牢记一些推荐做法，它们有助于简化故障排除的流程。下方列出了一些Playbook开发推荐做法：
- 使用play或任务的用途的简要描述来命名play和任务。在执行Playbook时，play名称或任务名称会显示出来。这也有助于记录每个play或任务应该达到的目标，以及可能需要它的原因
- 包含注释，以添加与任务相关的其他内嵌文档
- 有效利用垂直空白。通常，垂直组织任务属性可以使它们更易于阅读
- 一致的水平缩进至关重要。使用空格而不是制表符，以避免缩进错误。将文本编辑器设置为当按下`Tab`键时插入空格，以简化操作
- 尽可能使Playbook简单，仅使用需要的功能

## 对Ansible受管主机进行故障排除
### 将检查模式用作测试工具
&#8195;&#8195;可以使用`ansible-playbook --check`命令对Playbook运行烟雾测试。此选项会执行 Playbook，但不对受管主机的配置进行更改。如果Playbook中使用的模块支持检查模式，则将显示已在受管主机上进行的更改，但不执行它们。如果模块不支持检查模式，则不显示更改，但模块仍然不执行任何操作：
```
[ansible@redhat9 ~]$ ansible-playbook --check file2.yml
PLAY [Change a file Test] **************************************************************************
TASK [SELinux type is set to samba_share_t] **************************************************************************
ok: [redhat8]
PLAY RECAP ***************************************************************
redhat8 : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```
注意事项：
- 如果任务使用了条件，则`ansible-playbook --check`命令可能无法正常运作

还可以通过`check_mode`设置控制各个任务是否以检查模式运行：
- 如果任务设置了`check_mode: yes`，则它始终以检查模式运行，不论是否将`--check`选项传递给`ansible-playbook`
- 如果任务设置了`check_mode: no`，它将始终照常运行，即使将`--check`传递给`ansible-playbook`

以下任务始终以检查模式运行，并且不进行更改：
```yml
  tasks:
    - name: task always in check mode
      shell: uname -a
      check_mode: yes
```
以下任务始终正常运行，即使其开头是`ansible-playbook --check`：
```yml
  tasks:
    - name: task always runs even in check mode
      shell: uname -a
      check_mode: no
```
`check_mode`用处：
- 可以在利用`check_mode: yes`测试个别任务的过程中正常运行Playbook中的大部分任务
- 也可利用`check_mode: no`运行选定的任务来收集事实或设置条件变量，但不更改受管主机，从而提高检查模式中试运行提供合理结果的几率

&#8195;&#8195;通过测试魔法变量`ansible_check_mode`的值，任务可以判断Playbook是否以检查模式运行。如果Playbook以检查模式运行，则此布尔值变量设为`true`。   
注意事项：
- 即使通过`ansible-playbook --check`运行Playbook，设置了`check_mode: no`的任务也将运行。因此，如果不通过检查Playbook及其关联的任何角色或任务来进行确认，就无法确信`--check`选项不会更改受管主机。
注意
- 如果有较旧的 Playbook并且它们使用`always_run: yes`在检查模式中强制照常运行任务，那么在Ansible 2.6和更新版本中，必须要将这些代码替换为`check_mode: no`

&#8195;&#8195;`ansible-playbook`命令还提供一个`--diff`选项。此选项可报告对受管主机上的模板文件所做的更改。如果与`--check`选项结合，则命令输出中会显示这些更改，但实际上不进行更改：
```
[student@demo ~]$ ansible-playbook --check --diff playbook.yml
```
### 使用模块进行测试
&#8195;&#8195;一些模块可以提供关于受管主机状态的额外信息。下面列出了一些Ansible模块，可用于测试和调试受管主机上的问题：
- `uri`模块提供了一种方式，可以检查`RESTful API`是否返回需要的内容：
    ```yml
      tasks:
    - uri:
        url: http://api.myapp.com
        return_content: yes
      register: apiresponse

    - fail:
        msg: 'version was not provided'
      when: "'version' not in apiresponse.content"
    ```
- `script`模块支持在受管主机上执行脚本，如果该脚本的返回代码不是零，则报告失败。脚本必须存在于控制节点上，传输到受管主机并在其上执行：
    ```yml
      tasks:
    - script: check_free_memory
    ```
- `stat`模块收集文件的事实，与`stat`命令非常相似。可以使用它来注册变量，然后进行测试来确定文件是否存在或获取有关该文件的其他信息。如果文件不存在，则`stat`任务不会失败，但其注册的`*.stat.exists`变量会报告为`false`。下面示例中，`/var/run/app.lock`存在时应用仍然会运行；这时，play应该会中止：
    ```yml
      tasks:
    - name: Check if /var/run/app.lock exists
      stat:
        path: /var/run/app.lock
      register: lock

    - name: Fail if the application is running
      fail:
      when: lock.stat.exists
    ```
- `assert`是`fail`模块的一种替代选择。`assert`模块支持`that`选项，该选项取一个条件列表作为值。如果这些条件中的任何一个为`false`，则任务失败。可以使用`success_msg`和`fail_msg`选项来自定义它报告成功或失败时显示的消息。以下示例重复上一个示例，但使用`assert`来替代`fail`：
    ```yml
      tasks:
    - name: Check if /var/run/app.lock exists
      stat:
        path: /var/run/app.lock
      register: lock

    - name: Fail if the application is running
      assert:
        that:
          - not lock.stat.exists
    ```

### 对连接进行故障排除
&#8195;&#8195;使用Ansible管理主机时的许多常见问题与主机连接相关，也与围绕远程用户和特权升级的配置问题有关：
- 如果遇到与受管主机身份验证相关的问题，请确保在配置文件或play中正确设置了`remote_user`。还应确认设置了正确的`SSH`密钥或为该用户提供正确的密码
- 确保正确设置`become`，并使用正确的`become_user`（默认为root）。应该确认输入了正确的`sudo`密码并且受管主机上正确配置了`sudo`

一个更微妙的问题与清单设置有关：
- 对于具有多个网络地址的复杂服务器，连接该系统时可能需要使用特定的地址或DNS名称
- 可能不希望将该地址用作计算机的清单名称，从而提高可读性
- 可以设置主机清单变量`ansible_host`，它会用其他名称或IP地址覆盖清单名称并供Ansible用于连接该主机。该变量可以在该主机的`host_vars`文件或目录中设置，或者可在清单文件本身中设置

&#8195;&#8195;以下清单条目将Ansible配置为在处理主机`web4.phx.example.com`时连接到`192.0.2.4`，示例如下：
```
web4.phx.example.com ansible_host=192.0.2.4
```
这是控制Ansible如何连接受管主机的有用方式。但是，`ansible_host`的值不正确时也可能会导致问题。
### 使用临时命令测试受管主机
&#8195;&#8195;下面几个示例演示了一些可通过使用临时命令在受管主机上执行的检查。曾使用过`ping`模块来测试是否能够连接到受管主机。根据传递的选项，还可以使用它来测试是否正确配置了特权升级和凭据：
```
[ansible@redhat9 ~]$ ansible redhat8 -m ping
redhat8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
[ansible@redhat9 ~]$ ansible redhat8 -m ping --become
redhat8 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```
&#8195;&#8195;返回`demohost`受管主机中配置的磁盘上的当前可用空间。这可用于确认受管主机上的文件系统是否未满，示例如下：
```
[ansible@redhat9 ~]$ ansible redhat8 -m command -a 'df'
redhat8 | CHANGED | rc=0 >>
Filesystem            1K-blocks     Used Available Use% Mounted on
devtmpfs                 909368        0    909368   0% /dev
tmpfs                    924716        0    924716   0% /dev/shm
tmpfs                    924716     1392    923324   1% /run
tmpfs                    924716        0    924716   0% /sys/fs/cgroup
/dev/mapper/rhel-root  37734400 14083272  23651128  38% /
/dev/nvme0n1p1          1038336   172968    865368  17% /boot
/dev/nvme0n2p5           688048      568    651644   1% /testfs
tmpfs                    184940       16    184924   1% /run/user/42
tmpfs                    184940        4    184936   1% /run/user/1011
```
示例返回`demohost`受管主机上当前的可用内存：
```
[ansible@redhat9 ~]$ ansible redhat8 -m command -a 'free -m'
redhat8 | CHANGED | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:           1806         759         206           1         840        1015
Swap:          2047           0        2047
```
### 正确的测试等级
&#8195;&#8195;Ansible设计为可确保Playbook中包含的并由其模块执行的配置正确无误。它监控所有模块是否报告有故障，一旦遇到任何故障将立即停止Playbook。这有助于确保出现故障之前执行的任何任务都没有错误。  
&#8195;&#8195;正因为此，通常无需检查Ansible所管理的任务的结果是否已正确应用到受管主机上。当需要更直接的故障排除时，可以在Playbook中添加一些健康检查，或者以临时命令形式直接运行它们。但是，应该小心谨慎，不要为了重复检查模块本身执行的测试给为您的任务和play添加太多复杂性。
## 待补充