# AIX-常见维护

## 备份与恢复
### mksysb
#### mksysb制作引导光盘
&#8195;&#8195;系统备份mksysb也可以制作成引导光盘，可以用户系统重装或者引导进入维护模式，PowerVM下挂载虚拟光驱比较方便，步骤如下：
- 进入制作菜单：`# smitty mkdvd`
- `Use an existing mksysb image?`选项选择`Yes`
- `DVD backup media format?`选择`1 ISO9660(CD format)`,UDF格式应该也可以，本人用的不多
- 进入`Back Up This System to IS09660 DVD`菜单：
    - `Location of existing mksysb image`选项填入mksysb的路径及名称
    - `Remove final images after creating DVD?`选项填入`no`
    - `Create the DVD now?`选项填入`no`
- 回车开始制作，成功后会提示文件名称及存放路径，通常在`/mkcd/cd_images`下

注意事项：
- `/tmp`文件系统空间需要足够，否则会失败，报错示例：
    ```
    0301-152 bosboot: not enough file space to create:/tmp/cd.bi
    ```
- VG需要有足够的空间，过程中会创建多个临时文件系统，最终存放DVD的文件系统`/mkcd/cd_images`也是新建的。VG预留空间建议是mksysb的两倍左右

## 用户维护
### root用户
#### 重置root密码
&#8195;&#8195;忘记root密码后，如果某个可以登录在`security`组里面，可以尝试修改root密码，如果没有，只能通过引导光盘进入维护模式，示例使用光盘进入维护模式重设root密码步骤：
- 插入系统启动光盘（AIX安装光盘的CD 1，nim分发的系统或虚拟光驱，mksysb可以制作成iso），重启系统
- 在系统启动界面出现时，按`1`键，进入`sms menu`模式
- 选择`5（Select Boot Options)`
- 选择`1（Select Install /Boot Device`
- 选择`3（CD/DVD)`
- 选择`9（List all device）`
- 选择物理光驱或虚拟光驱（CD-ROM）
- 选择`2（Normal Mode Boot）`
- 选择`1（Yes）`
- 系统提示“STARTING SOFTWOARE PLEASE WAIT...”, 输入`1`选择系统终端
- 选择`1（Type 1 and press Enter to have English during install）`
- 选择`3（Start Maintenance Mode for System Receovery）`
- 选择`1（Access a Root Volume Group）`
- 选择`0（Continue）`
- 选择`1（Root Volume Group）`,PowerVM 环境可能为2，注意查看VG信息
- 选择`1（Access this Volume Group and start a shell）`，进入root用户提示符`#`
- 执行passwd，输入密码及再次确认密码，重启系统
- 系统启动完成，使用root用户和修改后的密码登录
- 取出系统光盘，密码修改完成

## 文件系统
### 
## 待补充