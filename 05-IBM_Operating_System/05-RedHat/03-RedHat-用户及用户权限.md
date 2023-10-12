# RedHat-用户及用户权限笔记
使用RedHat8系统学习时候的一些基础学习笔记。
## 用户及用户权限
### 用户及用户组创建
#### 创建用户组和组
创建名为ftpusers的用户组，组id为4000：
```
[root@redhat8 ~]# groupadd -g 40000 ftpusers
[root@redhat8 ~]# cat /etc/group |sed -n '/ftpusers/p'
ftpusers:x:40000:
```
创建名为christ和harry的用户，并将ftpusers作为其附属组：
```
[root@redhat8 ~]# useradd -G ftpusers christ
[root@redhat8 ~]# id christ
uid=1003(christ) gid=1003(christ) groups=1003(christ),40000(ftpusers)
[root@redhat8 ~]# useradd -G ftpusers harry
[root@redhat8 ~]# id harry
uid=1005(harry) gid=1005(harry) groups=1005(harry),40000(ftpusers)
[root@redhat8 ~]# cat /etc/group |sed -n '/ftpusers/p'
ftpusers:x:40000:christ,harry
```
&#8195;&#8195;默认情况`useradd`时候会给用户创建一个同名组，如果需要属于其它组，那么使用`-G`来指定附属组。使用`-g`表示创建用户的时设置用户属于主要的用户组，所以不会创建同名组，不指定`-g`就会创建同名组：
```
[root@redhat8 ~]# useradd -g christ christb
[root@redhat8 ~]# id christb
uid=1004(christb) gid=1003(christ) groups=1003(christ)
```
创建名为sarah的用户，不属于ftpusers组，并设定为不可登录shell：
```
[root@redhat8 ~]# useradd -s /sbin/nologin sarah
[root@redhat8 ~]# id sarah
uid=1006(sarah) gid=1006(sarah) groups=1006(sarah)
[root@redhat8 ~]# cat /etc/passwd |grep sarah
sarah:x:1006:1006::/home/sarah:/sbin/nologin
[root@redhat8 ~]# su - sarah
This account is currently not available.
[root@redhat8 ~]# sudo -u sarah ls /home/sarah
[root@redhat8 ~]# 
```
说明：
- 命令`su`表示切用户，后面`-`表示切换用户后套用用户的环境设置
- 命令`sudo -u sarah ls /home/sarah`表示使用sarah用户去执行命令`ls /home/sarah`

将上述用户密码均设置为test1234,两种方法示例如下：
```
[root@redhat8 ~]# passwd christ
Changing password for user christ.
New password: 
BAD PASSWORD: The password fails the dictionary check - it is too simplistic/systematic
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@redhat8 ~]# echo "test1234" |passwd harry --stdin
Changing password for user harry.
passwd: all authentication tokens updated successfully.
```
说明：`echo`方式不建议在命令行使用，因为命令`history`可以看到设置的密码。
#### useradd命令
useradd命令常用参数如下：
- `-b, --base-dir BASE_DIR`：新用户主目录的基本目录
- `-c, --comment COMMENT`：新用户的GECOS字段 
- `-d，--home-dir HOME_DIR`：指定新用户的home目录
- `-D, --defaults`：打印或更改默认useradd配置
- `-e, --expiredate EXPIRE_DATE`：新用户的过期日期
- `-f, --inactive INACTIVE`：新帐户的密码不活动
- `-g, --gid GROUP`：新用户的首要组名或组ID
- `-G, --groups GROUPS`:新帐户的补充组列表
- `-h, --help`：显示帮助信息并退出
- `-k, --skel SKEL_DIR`：使用此备用框架目录
- `-K, --key KEY=VALUE`：覆盖`/etc/login.defs`默认值
- `-l, --no-log-init`：不要将用户添加到lastlog和faillog数据库
- `-m, --create-home`：创建新用户的home目录
- `-M, --no-create-home`：不创建新用户的home目录
- `-N, --no-user-group`：不创建与用户同名的组
- `-o, --non-unique`：允许创建具有重复（非唯一）UID的用户
- `-p, --password PASSWORD`：新用户的加密密码（/etc/shadow里面密码）
- `-r, --system`：创建系统帐户
- `-R, --root CHROOT_DIR`：要chroot到的目录
- `-P, --prefix PREFIX_DIR`：`/etc/*`文件所在的前缀目录
- `-s, --shell SHELL`：新用户的登录shell
- `-u, --uid UID`：新用户的用户ID
- `-U, --user-group`：创建和用户同名的组
- `-Z, --selinux-user SEUSER`：对SELinux用户映射使用特定的SEUSER

### 用户权限
创建`/var/ftp/pub`目录，按以下要求配置目录权限：
- `/var/ftp/pub`目录的所有者为root
- `/var/ftp/pub`目录的所属组为root，任何用户对pub目录下的文件有读权限，没有写权限
- 创建`/var/ftp/pub/christ`目录，所有者为christ，所属组为ftpusers，ftpusers组用户对目录下文件有读权限，没有写权限，其它用户读写权限都没有并不可见
- 创建`/var/ftp/pub/harry`目录，所有者为harry，所属组为ftpusers，ftpusers组用户对目录下文件有读权限，没有写权限其它用户读写权限都没有并不可见
- sarah用户对`/var/ftp/pub/harry/project`目录有读写权限，对其它目录和文件没有读写权限

创建`/var/ftp/pub`目录：
```
[root@redhat8 ~]# mkdir /var/ftp/pub
[root@redhat8 ~]# ls -l /var/ftp
total 0
drwxr-xr-x. 2 root root 6 Oct 28 02:01 pub
```
创建`/var/ftp/pub/christ`目录并修改权限：
```
[root@redhat8 ~]# mkdir /var/ftp/pub/christ
[root@redhat8 ~]# ls -l /var/ftp/pub
total 0
drwxr-xr-x. 2 root root 6 Oct 28 03:51 christ
[root@redhat8 ~]# chown christ.ftpusers /var/ftp/pub/christ
[root@redhat8 ~]# ls -l /var/ftp/pub
total 0
drwxr-xr-x. 2 christ ftpusers 6 Oct 28 03:51 christ
[root@redhat8 ~]# chmod 750 /var/ftp/pub/christ
[root@redhat8 ~]# ls -l /var/ftp/pub
total 0
drwxr-x---. 2 christ ftpusers 6 Oct 28 03:51 christ
[root@redhat8 ~]# chmod o+r /var/ftp/pub/christ
[root@redhat8 ~]# ls -l /var/ftp/pub
total 0
drwxr-xr--. 2 christ ftpusers 6 Oct 28 03:51 christ
[root@redhat8 ~]# chmod o-r /var/ftp/pub/christ
[root@redhat8 ~]# ls -l /var/ftp/pub
total 0
drwxr-x---. 2 christ ftpusers 6 Oct 28 03:51 christ
[root@redhat8 ~]# sudo -u sarah ls /var/ftp/pub
christ
```
创建一个文件的进行测试验证及修改christ目录：
```
[root@redhat8 ~]# touch /var/ftp/pub/christ/test
[root@redhat8 ~]# ls -l /var/ftp/pub/christ/test
-rw-r--r--. 1 root root 0 Oct 28 04:31 /var/ftp/pub/christ/test
[root@redhat8 ~]# chown christ.ftpusers /var/ftp/pub/christ -R
[root@redhat8 ~]# ls -l /var/ftp/pub/christ/test
-rw-r--r--. 1 christ ftpusers 0 Oct 28 04:31 /var/ftp/pub/christ/test
[root@redhat8 ~]# sudo -u sarah ls /var/ftp/pub/christ
ls: cannot open directory '/var/ftp/pub/christ': Permission denied
[root@redhat8 ~]# sudo -u sarah ls /var/ftp/pub/christ/test
ls: cannot access '/var/ftp/pub/christ/test': Permission denied
[root@redhat8 ~]# chmod o-r /var/ftp/pub/christ/test
[root@redhat8 ~]# ls -l /var/ftp/pub/christ/test
-rw-r-----. 1 christ ftpusers 0 Oct 28 04:31 /var/ftp/pub/christ/test
[root@redhat8 ~]# sudo -u harry cat /var/ftp/pub/christ/test
test txt
[root@redhat8 ~]# sudo -u sarah cat /var/ftp/pub/christ/test
cat: /var/ftp/pub/christ/test: Permission denied
```
创建harry目录并修改权限：
```
[root@redhat8 ~]# mkdir -p /var/ftp/pub/harry/project
[root@redhat8 ~]# chown harry.ftpusers /var/ftp/pub/harry -R
[root@redhat8 ~]# ls -l /var/ftp/pub
total 0
drwxr-x---. 2 christ ftpusers 18 Oct 28 04:41 christ
drwxr-xr-x. 3 harry  ftpusers 21 Oct 28 04:43 harry
[root@redhat8 ~]# chmod o-rx /var/ftp/pub/harry -R
[root@redhat8 ~]# ls -l /var/ftp/pub
total 0
drwxr-x---. 2 christ ftpusers 18 Oct 28 04:41 christ
drwxr-x---. 3 harry  ftpusers 21 Oct 28 04:43 harry
[root@redhat8 ~]# sudo -u harry echo 123 > /var/ftp/pub/harry/project/test
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project
total 4
-rw-r--r--. 1 root root 4 Oct 28 04:49 test
[root@redhat8 ~]# chown harry.ftpusers /var/ftp/pub/harry -R
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project
total 4
-rw-r--r--. 1 harry ftpusers 4 Oct 28 04:49 test
[root@redhat8 ~]# chmod o-rx /var/ftp/pub/harry -R
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project
total 4
-rw-r-----. 1 harry ftpusers 4 Oct 28 04:49 test
```
sarah用户对`/var/ftp/pub/harry/project`目录有读写权限:
```
[root@redhat8 ~]# sudo -u christ cat /var/ftp/pub/harry/project/test
123
[root@redhat8 ~]# sudo -u sarah cat /var/ftp/pub/harry/project/test
cat: /var/ftp/pub/harry/project/test: Permission denied
[root@redhat8 ~]# sudo -u sarah ls -l /var/ftp/pub
total 0
drwxr-x---. 2 christ ftpusers 18 Oct 28 04:41 christ
drwxr-x---. 3 harry  ftpusers 21 Oct 28 04:43 harry
[root@redhat8 ~]# setfacl -R -m u:sarah:rw- /var/ftp/pub/harry/project
[root@redhat8 ~]# sudo -u christ cat /var/ftp/pub/harry/project/test
123
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project
total 4
-rw-rw----+ 1 harry ftpusers 4 Oct 28 04:49 test
```
说明：
- 权限中多了个`+`表示有扩充权限,`ls`是列不出来的
- 使用`gesfacl`可以单独给某个用户加权限，加了之后发现还不行，上层目录的权限一样也要改

示例如下：
```
[root@redhat8 ~]# getfacl /var/ftp/pub/harry/project
getfacl: Removing leading '/' from absolute path names
# file: var/ftp/pub/harry/project
# owner: harry
# group: ftpusers
user::rwx
user:sarah:rw-
group::r-x
mask::rwx
other::---
[root@redhat8 ~]# sudo -u sarah cat /var/ftp/pub/harry/project/test
cat: /var/ftp/pub/harry/project/test: Permission denied
[root@redhat8 ~]# setfacl -R -m u:sarah:rwx /var/ftp/pub/harry
[root@redhat8 ~]# sudo -u sarah cat /var/ftp/pub/harry/project/test
123
```
### 用户和权限示例
#### setgid属性的作用
&#8195;&#8195;使用`sudo -u`命令以sarah的身份在`/var/ftp/pub/harry/project/`目录下建立一个名为`sarah`的子目录，使用ls -l命令查看子目录的权限位显示：
```
[root@redhat8 ~]# sudo -u sarah mkdir /var/ftp/pub/harry/project/sarah
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project
total 4
drwxr-xr-x. 2 sarah sarah    6 Oct 28 07:34 sarah
-rw-rwx---+ 1 harry ftpusers 4 Oct 28 04:49 test
```
&#8195;&#8195;使用命令`chmod g+s`对`sarah`目录操作，使用ls -l命令查看子目录的权限位显示:
```
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project
total 4
drwxr-sr-x. 2 sarah sarah    6 Oct 28 07:34 sarah
-rw-rwx---+ 1 harry ftpusers 4 Oct 28 04:49 test
```
&#8195;&#8195;以root用户建立`/var/ftp/pub/harry/project/sarah/folder1`，使用`ls -l`命令查看子目录的所属组:
```
[root@redhat8 ~]# mkdir /var/ftp/pub/harry/project/sarah/folder1
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project/sarah
total 0
drwxr-sr-x. 2 root sarah 6 Oct 28 07:39 folder1
```
&#8195;&#8195;以root用户新建`/var/ftp/pub/harry/project/sarah/file1`，使用`ls -l`命令查看文件的所属组：
```
[root@redhat8 ~]# touch /var/ftp/pub/harry/project/sarah/file1
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project/sarah
total 0
-rw-r--r--. 1 root sarah 0 Oct 28 07:40 file1
drwxr-sr-x. 2 root sarah 6 Oct 28 07:39 folder1
```
&#8195;&#8195;以christ, harry用户尝试新建的`sarah`目录访问和操作:
```
[root@redhat8 ~]# sudo -u christ ls /var/ftp/pub/harry/project/sarah/file1
/var/ftp/pub/harry/project/sarah/file1
[root@redhat8 ~]# sudo -u harry ls /var/ftp/pub/harry/project/sarah/file1
/var/ftp/pub/harry/project/sarah/file1
[root@redhat8 ~]# sudo -u harry ls /var/ftp/pub/harry/project/sarah/folder1
[root@redhat8 ~]# sudo -u christ ls /var/ftp/pub/harry/project/sarah/folder1
```
#### setuid属性的作用
&#8195;&#8195;将touch命令文件`cp`过来，并使用ls -l 命令查看目标mytouch文件的权限位、所有者（sarah），所属组(ftpuser或sarah)：
```
[root@redhat8 ~]# sudo -u sarah cp /usr/bin/touch /var/ftp/pub/harry/project/mytouch 
[root@redhat8 ~]# ls -l /var/ftp/pub/harry/project
total 112
-rwxr-xr-x. 1 sarah sarah    109776 Oct 28 08:43 mytouch
drwxr-sr-x. 3 sarah sarah        34 Oct 28 07:40 sarah
-rw-rwx---+ 1 harry ftpusers      4 Oct 28 04:49 test
```
尝试使用mytouch：
```
[root@redhat8 ~]# cd /var/ftp/pub/harry/project
[root@redhat8 project]# sudo -u sarah ./mytouch sarah1
[root@redhat8 project]# ls -l sarah1
-rw-r--r--. 1 sarah sarah 0 Oct 28 08:52 sarah1
[root@redhat8 project]# sudo -u christ ./mytouch christ1
./mytouch: cannot touch 'christ1': Permission denied
```
修改权限后再次尝试示例：
```
[root@redhat8 project]# chmod u+s ./mytouch
[root@redhat8 project]# ls -l mytouch
-rwsr-xr-x. 1 sarah sarah 109776 Oct 28 08:43 mytouch
[root@redhat8 project]# sudo -u christ ./mytouch christ1
[root@redhat8 project]# ls -l 
total 112
-rw-r--r--. 1 sarah christ        0 Oct 28 08:56 christ1
-rwsr-xr-x. 1 sarah sarah    109776 Oct 28 08:43 mytouch
drwxr-sr-x. 3 sarah sarah        34 Oct 28 07:40 sarah
-rw-r--r--. 1 sarah sarah         0 Oct 28 08:52 sarah1
-rw-rwx---+ 1 harry ftpusers      4 Oct 28 04:49 test
```
#### stickybit属性的作用
创建一个tmp目录修改权限后并查看权限位、所有者，所属组：
```
[root@redhat8 project]# mkdir /var/ftp/pub/tmp
[root@redhat8 project]# chmod 777 /var/ftp/pub/tmp
[root@redhat8 project]# ls -l /var/ftp/pub
total 0
drwxr-x---. 2 christ ftpusers 18 Oct 28 04:41 christ
drwxrwx---+ 3 harry  ftpusers 21 Oct 28 04:43 harry
drwxrwxrwx. 2 root   root      6 Oct 28 09:00 tmp
```
用harry用户创建一个文件harry1，查看权限位、所有者，所属组，并尝试用sarah用户删除：
```
[root@redhat8 project]# sudo -u harry touch /var/ftp/pub/tmp/harry1
[root@redhat8 project]# ls -l /var/ftp/pub/tmp
total 0
-rw-r--r--. 1 harry harry 0 Oct 28 09:01 harry1
[root@redhat8 project]# sudo -u sarah rm /var/ftp/pub/tmp/harry1
rm: remove write-protected regular empty file '/var/ftp/pub/tmp/harry1'? y       
[root@redhat8 project]# ls -l /var/ftp/pub/tmp
total 0
```
修改权限后，再次用harry用户创建文件harry2，查看权限位、所有者，所属组，并尝试用sarah用户删除：
```
[root@redhat8 project]# chmod a+t /var/ftp/pub/tmp
[root@redhat8 project]# ls -l /var/ftp/pub
total 0
drwxr-x---. 2 christ ftpusers 18 Oct 28 04:41 christ
drwxrwx---+ 3 harry  ftpusers 21 Oct 28 04:43 harry
drwxrwxrwt. 2 root   root      6 Oct 28 09:02 tmp
[root@redhat8 project]# sudo -u harry touch /var/ftp/pub/tmp/harry2
[root@redhat8 project]# ls -l /var/ftp/pub/tmp
total 0
-rw-r--r--. 1 harry harry 0 Oct 28 09:03 harry2
[root@redhat8 project]# sudo -u sarah rm /var/ftp/pub/tmp/harry2
rm: remove write-protected regular empty file '/var/ftp/pub/tmp/harry2'? y
rm: cannot remove '/var/ftp/pub/tmp/harry2': Operation not permitted
```
说明：
- 命令`chmod u+s`设置用户setuid属性，表示针对修改的目录或文件对于任何用户都有读写这个的权限
- 命令`chmod g+s`设置组setgid属性，此权限表示文件是在拥有文件的组的权限下执行的，而不是在执行文件的用户的组的权限下执行的（没有setgid的情况）。如果应用于目录，则在目录中创建的所有文件都归拥有该目录的组所有，而不是由创建文件的用户组所拥有
- 命令`chmod o+t`设置其它用户stickybit属性，主要用于目录，该位指示目录中创建的文件只能由创建该文件的用户（或root）删除

英文原文描述：
- sticky bit：used primarily on directories, this bit dictates that a file created in the directory can be removed only by the user that created the file. It is indicated by the character t in place of the x in the everyone category. If the everyone category does not have execute permissions, the T is capitalized to reflect this fact.
- setuid：used only for binary files (applications), this permission indicates that the file is to be executed with the permissions of the owner of the file, and not with the permissions of the user executing the file (which is the case without setuid). This is indicated by the character s in the place of the x in the owner category. If the owner of the file does not have execute permissions, a capital S reflects this fact.
- setgid：used primarily for binary files (applications), this permission indicates that the file is executed with the permissions of the group owning the file and not with the permissions of the group of the user executing the file (which is the case without setgid). If applied to a directory, all files created within the directory are owned by the group owning the directory, and not by the group of the user creating the file. The setgid permission is indicated by the character s in place of the x in the group category. If the group owning the file or directory does not have execute permissions, a capital S reflects this fact. drwxr-sr-x. 4 sarah sarah 34 Oct 15 21:27 sarah
