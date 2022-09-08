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