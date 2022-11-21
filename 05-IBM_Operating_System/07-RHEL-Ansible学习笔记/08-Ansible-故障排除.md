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
