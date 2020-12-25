# AIX-用户策略
## 用户属性文件
AIX中每个用户都有相关属性，与用户属性相关的文件：
- 用户基本属性：/etc/passwd
- 用户环境属性：/etc/security/environ
- 定义用户的资源定额和限制：/etc/security/limits
- 用户的扩展属性：/etc/security/user
- 用户的管理角色属性：/etc/security/user.roles
- 审计配置信息：/etc/security/audit/config
- 组的基本属性：/etc/group
- 组的扩展属性：/etc/security/group
- 用户最后登录属性：/etc/security/lastlog

## 常用命令
常用用户操作命令如下：

命令|用途
:---|:---
lsuser -f root|以节格式查看用户属性
lsgroup -f system|以节格式查看用户组属性
mkgroup -a tmpg |新建名为tmpg的组管理账户
mkuser -a test |新建名为test的管理账户
rmgroup tmpg |删除用户组tmpg
rmuser -p test |删除用户test及所有属性
pwdadm test |为test设置密码
passwd |修改当前用户密码
chuser |修改用户属性

## 用户属性修改
创建修改用户都可以使用smit菜单，参数一目了然，推荐用此方法；不推荐直接修改配置文件，后果可能会很严重，用命令也要注意。
### 修改配置文件
修改/etc/security/user文件注意事项：
- 修改default的配置会影响所有用户，如果用户指定了相应的配置，以用户指定的配置为准
- 有些default的配置对root无效，例如maxexpired
- 修改expires参数要注意格式，此参数是设置用户到期时间，如果不慎修改了default里面的值，并且格式不对，例如`expires = 120`，那么所有未单独指定此值的用户（一般都是采用default）都将很快过期
- 修改maxage注意，如果修改的是default里面的值，会应用到所有未配置此值的用户，例如设为4，4周内没改过密码，那么此用户将在下一次登录提示更改密码
- 修改密码策略例如密码长度，需要字符数量等，会在下一次修改密码提示，目前的密码不受影响
- 在修改user文件后，用户间配置需要空一行，否则会出现问题

### 修改用户示例
修改用户test的到期日期为2020年12月31日12点：
```shell
chusr expires=1231120020 test
```
使用户test能够远程访问此系统：
```shell
chusr rlogin=true test
```

## 用户基本属性
以下是控制登录并且与密码质量无关的用户属性：
属性名称|属性说明
:---|:---
account_locked|如果明确地需要锁定账户，那么该属性可以设置为True；缺省值为False
admin|如果设置为True，那么该用户无法更改密码。只有管理员可以更改它。
admgroups|列出子用户具有管理权限的组。对于这些组，该用户可以添加或删除成员。
auth1 |用于授权用户访问的认证方式。不推荐使用
auth2|按auth1指定的无论上面对用户进行认证后的方法。它无法阻止对系统的访问。一般为None
daemon |此布尔参数指定是否允许用户使用startsrc命令启动守护程序或子系统，
login|指定是否运行该用户登录。
logintimes|限制用户何时可以登录。
registry|指定用户注册表。
rlogin|指定所指定用户能否使用rlogin或telnet命令登录
su|指定其它用户是否可以使用su命令切换至此标识。
sugroups|指定运行哪个组结合至此用户标识
ttys|限制某些账户进入物理安全区域
expires	|定义用户账号过期时间，用于管理学生或访客账户；也可以用于临时关闭账户
loginretries |指定用户标识被系统锁定之前连续的可以尝试登录失败的最大次数。
umask |指定用户的初始umask
rcmds|指定所指定用户能否使用rsh抿了或rexec命令运行各个命令。
hostallowedlogin|指定许可用户登录的主机。
hostdeniedlogin|指定不许可用户登录的主机
maxulogs|指定每个用户的最大登录数。

## 用户登录策略
建议用户首先使用自己的普通用户登录，然后允许su命令去登录到root，不建议以root用户的身份登录。下表是摘自IBM官方登录策略推荐：
操作名称|描述|推荐设置
:---|:---|:---
Interval between unsuccessful logins|用于为/etc/security/login.cfg中的logininterval属性设置相应的值，该参数的作用是指定一段时间间隔（以秒计），在该时间内，如果尝试进行了若干次登陆后仍无法成功登录，那么将禁用该端口。例如，如果logininterval设为60，logindisable设为4，那么如果在1分钟内发生4此尝试登录失败后就将禁用该账户|高安全性：300;中安全性：600;低安全性：无效;AIX标准设置：无限制
Number of login attempts before locking the account|用于为/etc/security/login.cfg中的loginrteries属性设置相应的值，该属性的作用是指定每个账户上可连接进行的登录尝试次数，如果经过在达到该次数后仍无法登录，那么将禁用相应的账户。不要对root账户设置该属性|高安全性：3;中安全性：4;低安全性：5;AIX标准设置：无限制
Remote root login|用于更改/etc/security/user中的rlogin属性设置相应的值，该属性的作用是指定系统上是否允许以root账户远程登录|	高安全性：False;中安全性：False;低安全性：无效;AIX标准设置：True
Re-enable login after locking|用于为/etc/security/login.cfg中的loginreenable属性设置相应的值，该属性的作用是指定一段时间间隔（以秒计），当端口被logindisable禁用后，经过此指定的时间间隔后将解锁端口|高安全性：360;中安全性：30;低安全性：无效;AIX标准设置：无限制
Disable login after unsuccessful login attempts	|用于为/etc/security/login.cfg中的logindisabel属性设置相应的值，该属性的作用是指定端口锁定前可进行的登录尝试次数|高安全性：10;中安全性：10;低安全性：无效;AIX标准设置：无限制
Login timeout|用于为/etc/security/login.cfg中的ligintimeout属性设置相应的值，该属性的作用是指定输入密码时允许的时间间隔|高安全性：30;中安全性：60;低安全性：60;AIX标准设置：60
Delay between unsuccessful logins|用于为/etc/security/login.cfg中的logindelay属性设置相应的值，该属性的作用是指定两次失败登录之间的延迟时间（以秒计）。每次登录失败后将附加一段延迟时间。例如，设置为5，在第一次登录失败后，终端将等待5秒，然后再发出下一次登录请求，第二次失败后，终端将等待10秒（2*5）|高安全性：10;中安全性：4;低安全性：5;AIX标准设置：无限制
Local login	|用于更改/etc/security/user中的login属性设置相应的值，该属性的作用是指定系统上是否允许以root账户登录控制台|高安全性：False;中安全性：无效;低安全性：无效;AIX标准设置：True

## 用户密码策略

良好的密码是抵御未授权进入系统的第一道有效防线。
符合以下条件的密码有效：
- 大小写字母的混合
- 字母、数字或标点符号的组合
- 未写在任何地方
- 如果使用/etc/security/passwd文件，那么长度最少未7个字符最大PW_PASSLEN个字符
- 不是在字典中可以查到的真实单词
- 不是键盘上的字母的排列模式，例如qwerty
- 不是真实单词或已知排列模式的反向拼写
- 不包含任何与您自己、家庭或朋友有关的个人信息
- 不与从前一个密码的模式相同
- 可以较快输入，这样便是的人就不能确定您的密码

除了这些机制外，还可以通过对密码进行限制，使其不得包含可能被猜到的标准UNIX单词，从而实施更严格的规则。
下表是摘自IBM官方密码策略规则推荐：
操作名称|描述|推荐设置
:---|:---|:---
Minimum number of characters|用于为/etc/security/user中的mindiff属性设置相应的值，该属性的作用是指定组成新密码必须的最少字符|高安全性：4；中安全性：3；低安全性：无效；AIX标准设置：无限制
Minimum age of characters|用于为/etc/security/user中的minage属性设置相应的值，该属性的作用是指密码可更改前必须经过的最小周数|	高安全性：1；中安全性：4；低安全性：无效；AIX标准设置：无限制
Maximum age of characters|用于为/etc/security/user中的maxage属性设置相应的值，该属性的作用是指密码可更改前必须经过的最大周数|	高安全性：13；中安全性：13；低安全性：52；AIX标准设置：无限制
Minimum length for password|用于为/etc/security/user中的minlen属性设置相应的值，该属性的作用是指定密码的最小长度|	高安全性：8；中安全性：8；低安全性：8；AIX标准设置：无限制
Minimum number of alphabetic characters	|用于为/etc/security/user中的minalpha属性设置相应的值，该值指定密码中包含的字母字符的最小数目|高安全性：2；中安全性：2；低安全性：2；AIX标准设置：无限制
Password reset time|用于为/etc/security/user中的histexpire属性设置相应的值，该属性的作用是指定密码可以重置前必须经过的周数。|	高安全性：13；中安全性：13；低安全性：26；AIX标准设置：无限制
Maximum times a char can appear in a password|用于为/etc/security/user中的maxrepeats属性设置相应的值，该属性的作用是指定密码中同样字符运行出现的最大次数|高安全性：2；中安全性：无效；低安全性：无效；AIX标准设置：8
Password reuse time	|用于为/etc/security/user中的histsize属性设置相应的值，该属性的作用是指定用户不能符用的旧密码个数|高安全性：20；中安全性：4；低安全性：4；AIX标准设置：无限制
Time to change pass-word after the expire-tion|用于为/etc/security/user中的maxexpired属性设置相应的值，该属性的作用是指定maxage后用户可更改到期密码的最长周期	|高安全性：2；中安全性：4；低安全性：8；AIX标准设置：-1
Minimum number of non-alphabetic char-acters|用于为/etc/security/user中的minother属性设置相应的值，该属性的作用是指定密码中非字母字符的最少个数|高安全性：2；中安全性：2；低安全性：2；AIX标准设置：无限制
Password expiration warning time|用于为/etc/security/user中的pwdwarntime属性设置相应的值，该属性的作用是指定系统发出需要更改密码的警告前等待的天数|高安全性：5；中安全性：14；低安全性：5；AIX标准设置：无限制

## 修改root用户注意事项
### 修改/etc/security/user文件注意
&#8195;&#8195;最近改了一些系统root用户的参数，都是直接修改/etc/security/user文件，主要是root用户过期时间，即参数`maxage = 15`，表示15周过期。过一段时间发现登录root提示：Your account has expired。     
找了几个测试分区，都是很久没改过，修改`maxage = 15`后，有的提示：
```
[compat]: 3004-332 Your password has expired.
```
有的分区提示：
```
3004-302 Your account has expired; please see the system administrator.
```
提示用户过期的改了密码是也是一样的提示。继续进行了一些测试：     
- 修改近期改过密码分区的root用户`maxage = 15`后（和下一个用户条目没有空行），登录root用户过期，删掉此行（删掉后和下一个用户条目间没有空行），重新试一样提示用户过期登录不了
- 修改近期改过密码分区的root用户`maxage = 15`后（和下一个用户条目有空行）,用户可以正常登录，密码也没过期；删掉空行，提示用户过期；加上空行，用户可以正常登录
- 修改长时间没改过root密码分区的root用户`maxage = 15`后（和下一个用户条目没有空行），会提示用户过期，无法进行登录
- 修改长时间没改过root密码分区的root用户`maxage = 15`后（和下一个用户条目有空行），会提示密码过期，也会提示输入新密码进行修改

&#8195;&#8195;注意，在修改过程中，保持一个root用户会话的连接，因为提示root用户过期后，没有其它系统配置免密登录的是无法再用root进入到系统的，只能通过光盘引导进入维护模式修改user文件，例如加空行可以使用下面命令：
```
sed '/daemon:/{x;p;x;}' user > user.bak
mv user.bak user
```
&#8195;&#8195;在操作user前建议备份一份，因为我第一次在使用命令`sed '/daemon:/{x;p;x;}' user > user`直接重定向到user文件后，user文件内容清空了，还好有备份。维护模式下vi虽然很不好用，但是依然可以vi文件然后`i`进行插入文本。
## 待补充
