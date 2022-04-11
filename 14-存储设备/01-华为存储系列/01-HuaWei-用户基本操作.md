# 用户及密码
华为存储用户相关操作。
## 默认登录方式
### OceanStor 5310 V5管理口
端口：每个控制器上的管理端口   
默认IP：A控192.168.128.101  B控192.168.128.102   
登录地址：https://192.168.128.102:8088   
默认用户：admin   
默认密码：Admin@Storage   
### 修改管理IP
以OceanStor 5310 V5为例：
- 选择“系统 > 控制器CTE0 > 控制器A或B > 管理端口
- 系统标记管理端口，鼠标指向管理端口，左键单击列出管理端口信息，点击右下角“修改“选项
  
## 密码重置方式
###  非超级用户重置密码
&#8195;&#8195;当管理员或只读用户忘记密码时，超级管理员可以通过DeviceManager或者CLI管理界面重置当前用户的密码。通过DeviceManager初始化用户密码（LDAP用户不能初始化密码）：
- 以超级管理员身份登录DeviceManager。系统初始的超级管理员用户名为`admin`，密码为`Admin@storage`
- 选择“设置 > 权限设置 > 用户管理”
- 在中间信息展示区，选择需要初始化密码的用户名并单击“修改”
- 系统弹出“修改用户”对话框。
- 选择“初始化密码”。输入“当前登录用户密码”、“新密码”和“确认密码”。
- 单击“确定”
- 系统弹出“执行结果”提示框，提示操作成功
- 单击“关闭”。

通过CLI管理界面重置用户密码。
- 以超级管理员身份登录CLI管理界面
- 执行`change user user_name=? action=reset_password`命令，重置用户的登录密码

示例重置“testuser”用户的登录密码：
```
admin:/>change user user_name=testuser action=reset_password
New password:**********
Reenter password:**********
Password:**********
Command executed successfully.
```
### 超级管理员忘记密码。
&#8195;&#8195;如果默认用户名为“admin”的超级管理员忘记密码，需使用“_super_admin”超级管理员通过串口登录CLI管理界面，执行`initpasswd`命令重置密码：
- 以用户名为`_super_admin`的超级管理员身份通过串口登录CLI管理界面。用户名为`_super_admin`的“超级管理员”，系统初始密码为`Admin@revive`。
- 执行`initpasswd`命令，重置超级管理员的登录密码。
    ```
    Storage: _super_admin> initpasswd
    please input username:admin
    init admin passwd,wait a moment please...
    *****please enter new password for admin :*****
    *****please re-enter new password for admin :*****
    Init admin passwd succeeded
    ```

### 待补充