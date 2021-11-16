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

### 保存和删除审计日志接收器