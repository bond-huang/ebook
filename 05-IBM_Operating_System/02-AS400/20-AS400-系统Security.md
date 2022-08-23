# AS400-系统Security
官方参考链接：[IBM i 7.3 Security](https://www.ibm.com/docs/zh/i/7.3?topic=security)
## Auditing security
官方参考链接：[Auditing security on IBM i](https://www.ibm.com/docs/zh/i/7.3?topic=reference-auditing-security-i)
### 使用Security Audit Journal
官方参考链接：[Using the security audit journal](https://www.ibm.com/docs/zh/i/7.3?topic=i-using-security-audit-journal)
#### CHGSECAUD设置安全审计
&#8195;&#8195;使用`CHGSECAUD`命令来激活系统安全审计(如果它不存在，将创建日志和日志接收器)、将`QAUDCTL`系统值设置为`*AUDLVL`并将`QAUDLVL`系统值设置为默认值。默认设置包括`*AUTFAIL`、`*CREATE`、`*DELETE`、`*SECURITY` 和 `*SAVRST`：
```
CHGSECAUD QAUDCTL(*AUDLVL) QAUDLVL(*DFTSET)
```
#### 设置安全审计
要设置安全审计步骤如下(需要`*AUDIT`特殊权限)：
1. 使用命令`CRTJRNRCV`在选择的库(示例使用JRNLIB)中创建日志接收器：
    ```
    CRTJRNRCV JRNRCV(JRNLIB/AUDRCV0001)
        THRESHOLD(100000) AUT(*EXCLUDE)
        TEXT('Auditing Journal Receiver')
    ```
    - 将日志接收器放在定期保存的库中。不要放在库`QSYS`中，即使它是日志所在的位置
    - 选择可用于后续方便命名的名称，例如`AUDRCV0001`。当更改日志接收器以继续命名时，可使用`*GEN`选项。择让系统管理更改用户的日志接收器时，使用这种类型的命名约定非常有帮助
    - 指定适合用户的系统大小和活动的接收器阈值。选择的大小应基于系统上的事务数和用户选择审核的操作数。如果使用系统change-journal management支持，日志接收器阈值必须至少为100000KB。更多信息可以参考[Journal management](https://www.ibm.com/docs/zh/i/7.3?topic=management-journal)
    - 在`AUT`参数上指定`*EXCLUDE`以限制对存储在日志中的信息的访问
2. 使用`CRTJRN`命令创建`QSYS/QAUDJRN`日志：
    ```
    CRTJRN JRN(QSYS/QAUDJRN) +
        JRNRCV(JRNLIB/AUDRCV0001) +
        MNGRCV(*SYSTEM) DLTRCV(*NO) +
        AUT(*EXCLUDE) TEXT('Auditing Journal')
    ```
    - `JRN`必须使用名称`QSYS/QAUDJRN`，`JRNRCV`指定在上一步中创建的日志接收器的名称
    - 在`AUT`参数上指定`*EXCLUDE`以限制对存储在日志中的信息的访问。必须有权将对象添加到`QSYS`
    - 使用`MNGRCV`(Manage receiver)参数让系统更改日志接收器并在附加的接收器超过创建日志接收器时指定的阈值时附加一个新接收器。如果选择此选项，则无需使用`CHGJRN`命令来分离接收器并手动创建和附加新接收器
    - 指定`DLTRCV(*NO)`(默认值）表示不要让系统删除分离的接收器。`QAUDJRN`接收器是用户的安全审计跟踪，在从系统中删除它们之前，确保它们被充分保存
3. 使用`WRKSYSVAL`命令设置`QAUDLVL`(audit level)系统值或`QAUDLVL2`(audit level extension)系统值。此两个值确定将哪些操作记录到系统上所有用户的审计日志中
4. 如有必要，使用`CHGUSRAUD`命令为单个用户设置操作审计，或为特定用户设置对象审计
5. 如有必要，使用`CHGOBJAUD`、`CHGAUD`和`CHGDLOAUD`命令为特定对象设置对象审计
6. 设置`QAUDENDACN`系统值以控制系统无法访问审计日志时发生的情况
7. 设置`QAUDFRCLVL`系统值以控制审计记录写入辅助存储的频率。参考防止丢失审计信息：[Preventing loss of auditing information](https://www.ibm.com/docs/zh/i/7.3?topic=auditing-preventing-loss-information#rzarlaudprv)
8. 通过将`QAUDCTL`系统值设置为`*NONE`以外的值来开始审计：
    - `QSYS/QAUDJRN`日志必须存在，然后才能将`QAUDCTL`系统值更改为`*NONE`以外的值
    - 当开始审计时，系统会尝试将记录写入审计日志。如果尝试不成功，会收到一条消息并且审核不会启动

### 管理审计日志和日志接收器
&#8195;&#8195;审计日志`QSYS/QAUDJRN`仅用于安全审计。不应将对象记录到审计日志中，例如使用`SNDJRNE`(Send Journal Entry)命令或`QJOSJRNE`(Send Journal Entry)API将用户条目发送到此日志。系统使用特殊的锁定保护来确保它可以将审计条目写入审计日志：
- 当审计处于活动状态时（`QAUDCTL`系统值不是`*NONE`），系统仲裁器作业`QSYSARB`会锁定`QSYS/QAUDJRN`日志
- 当审计处于活动状态时，不能对审计日志执行某些操作，例如：`DLTJRN`和`WRKJRN`命令，移动日志，恢复日志
- 审计日志中的所有安全条目都有一个日志代码`T`
- 除了安全条目之外，系统条目也出现在日志`QAUDJRN`中。这些是日志代码为`J`的条目，它们与初始程序加载 `IPL`和在日志接收器上执行的一般操作（例如，保存接收器）有关
- 如果日志或其当前接收器发生损坏，从而无法记录审计条目，则`QAUDENDACN`系统值将确定系统采取的操作

系统管理日志接收器的相关详细说明:
- 创建`QAUDJRN`日志时指定`MNGRCV(*SYSTEM)`，或修改为该值，系统会在达到阈值大小时自动分离接收器并创建及附加新的日志接收器，此功能叫做system change-journal management
- `QAUDJRN`中指定`MNGRCV(*USER)`，那么当日志接收器达到存储阈值时，消息将发送到为日志指定的消息队列。需使用`CHGJRN`命令分离接收器并附加一个新的日志接收器
- 日志的默认消息队列是`QSYSOPR`。也可以将不同的消息队列（例如 AUDMSG）:
    - 可以使用消息处理程序来监视`AUDMSG`消息队列。当收到日志阈值警告`CPF7099)`时，可以自动附加新的接收器
    - 如果您使用system change-journal management，那么当系统更改日志完成时，消息`CPF7020`将发送到日志消息队列，可以监视此消息，以便知道何时保存分离的日志接收器
- 使用Operational Assistant菜单时提供的自动清理功能不会清理`QAUDJRN`接收器。为避免磁盘空间问题，请定期分离、保存和删除`QAUDJRN`接收器
- 如果`QAUDJRN`日志不存在并且`QAUDCTL`系统值设置为非`*NONE`，则在`IPL`期间进行创建。这仅发生在异常情况之后，例如更换磁盘设备或清除辅助存储池

### 保存和删除审计日志接收器
附加一个新的审计日志接收器，保存并删除旧接收器：
- CHGJRN QSYS/QAUDJRN JRNRCV(*GEN)
- SAVOBJ (保存旧接收器)
- DLTJRNRCV (删除旧的接收器)

建议选择系统不忙的时间，并且应定期分离当前的审计日志接收器并附加一个新的接收器，原因：
- 如果每个日志接收器都包含特定、可管理时间段的条目，则分析日志条目会更容易
- 大型日志接收器会影响系统性能并占用辅助存储池上的宝贵空间

#### 系统管理的日志接收器
&#8195;&#8195;如果系统管理接收器，保存所有分离的`QAUDJRN`接收器并删除它们的步骤(使用日志消息队列和CPF7020消息的监视下述过程，CPF7020消息指示系统更改日志已成功完成)：
- 输入`WRKJRNA QAUDJRN`，显示当前连接的接收器，不要保存或删除此接收器
- 按`F15`到`work with the receiver directory`，显示与日志相关联的所有接收器及其相应的状态
- 使用`SAVOBJ`命令保存每个接收器，不要保存当前`attached`的接收器
- 使用`DLTJRNRCV`命令删除每个已经保存了的接收器

#### 用户管理的日志接收器
如果选择手动管理日志接收器，分离、保存和删除日志接收器过程：
- 输入`CHGJRN JRN(QAUDJRN) JRNRCV(*GEN)`命令，命令作业：
    - 分离当前连接的接收器
    - 使用下一个序列号创建一个新的接收器
    - 将新的接收器附加到日志
- 使用`WRKJRNA`命令显示当前连接了哪个接收器：WRKJRNA QAUDJRN
- 使用`SAVOBJ`命令保存分离的日志接收器，指定对象类型`*JRRNCV`
- 使用`DLTJRNRCV`命令删除接收器。如果删除没有保存的接收器，将收到一条警告消息

## Object Authority
### Object Authority for User
&#8195;&#8195;在对象属性中有用户对对象的权限。可以为用户分配几个不同的系统定义的对象权限级别，这些级别有如下几种：
- `*ALL`：允许对对象的所有操作，但仅限于所有者或受授权列表管理权限控制的操作除外
- `*CHANGE`：允许对对象的所有操作，但仅限于所有者或受对象存在权限、对象更改权限、对象引用权限和对象管理权限控制的操作
- `*EXCLUDE`：禁止对对象的所有操作
- `*USE`：允许访问对象属性和使用对象，但用户不能更改对象
- `USER DEF`：当特定对象权限和数据权限与上述任何预定义的对象权限级别不匹配时由系统显示。可以使用`display detail`功能键(F11)查看具体权限
- 在定义公共权限时`*AUTL`值也有效。表示该对象使用的授权列表中的公共权限规范

具体的对象权限有：
- `Opr`：对象操作权限提供查看对象属性和使用由用户对对象具有的数据权限指定的对象的权限
- `Mgt`：对象管理权限提供指定安全性、移动或重命名对象以及在对象是数据库文件时添加成员的权限
- `Exist`：对象存在权限提供控制对象存在和所有权的权限
- `Alter`：对象更改权限提供更改对象属性的权限，例如添加或删除触发器以及为数据库文件添加成员
- `Ref`：对象引用权限提供将对象指定为引用约束中的第一级的权限

具体的数据权限有：
- `Read`：读取权限提供访问对象内容的权限
- `Add`：添加权限提供向对象添加条目的权限
- `Update`：更新权限提供更改对象中现有条目内容的权限
- `Delete`：删除权限提供从对象中删除条目的权限
- `Execute`：执行权限提供运行程序或搜索库或目录的权限

### 待补充