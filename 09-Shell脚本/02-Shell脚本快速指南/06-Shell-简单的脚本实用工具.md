# Shell-简单的脚本实用工具
摘自书本《Linux命令行与shell脚本编程大全》中的一些简单的脚本实用工具。
## 归档
学习两种使用shell脚本备份Linux系统数据的方法。
### 归档数据文件
&#8195;&#8195;例如正在使用Linux系统作为一个重要项目的平台，可以创建一个shell来自动获取特定的目录的快照。在配置文件中指定所涉及的目录，在项目发生改变时，可以做出对应的修改。
#### 需要的功能
用tar命令创建工作目录归档文件示例：
```
[root@redhat8 tool]# tar -cf archive.tar /shell/gawk/*.*
tar: Removing leading `/' from member names
tar: Removing leading `/' from hard link targets
[root@redhat8 tool]# ls -l archive.tar
-rw-r--r--. 1 root root 10240 Dec 20 01:22 archive.tar
```
&#8195;&#8195;tar命令会显示一条告警消息，表明删除了路径名开头的斜线，将路径从绝对路径名变成相对路径名，这样可以将tar归档文件解压到文件系统中的任何地方。可以将STDERR重定向到`/dev/null`文件，就可以不显示了：
```
[root@redhat8 tool]# tar -cf archive.tar /shell/gawk/*.* 2>/dev/null
```
tar归档文件会消耗大量磁盘空间，建议压缩：
```
[root@redhat8 tool]# tar -zcf archive.tar.gz /shell/gawk/*.* 2>/dev/null
[root@redhat8 tool]# ls -l archive.tar.gz
-rw-r--r--. 1 root root 505 Dec 20 01:29 archive.tar.gz
```
可以将希望归档的每个目录或文件写入配置文件中：
```
[root@redhat8 tool]# cat Files_To_Backup
/shell/gawk
/shell/funciton
/python/jinja2
/python/fnmatch
```
&#8195;&#8195;可以让脚本读取配置文件，将会每个目录名加到归档列表中。使用read命令来读取该文件中的每一条记录即可，使用exec命令来重定向标准输入STDIN，用法如下：
```sh
exec < $CONFIG_FILE
read FILE_NAME
```
&#8195;&#8195;注意，为归档配置文件使用了一个变量：CONFIG_FILE，配置文件中每一条记录都会被读入，read发现还有可读记录，就会在?变量中返回一个表示成功的退出状态码0：
```sh
while [ $? -eq 0]
do
    [...]
read FILE_NAME
done
```
说明：
- read命令到了配置文件的末尾，就会返回一个非零状态码，就会退出while循环
- 在while中，必须将目录名加到归档列表中，还要检查那个目录是否存在

&#8195;&#8195;有时候会出现删除了一个目录却忘了更新归档配置文件的情况，可以使用if语句来检查目录是否存储，如果存在，就会加入到FILE_LIST中，如果没有就显示告警信息：
```sh
if [ -f $FILE_NAME -o -d $FILE_NAME ]
then
    # if file exists,add its name to the list.
    FILE_LIST="$FILE_LIST $FILE_NAME"
else
    # if file doernot exist,issue warning
    echo
    echo "$FILE_NAME,does not exist."
    echo "Obviously,I will not include it in this archive."
    echo "It is listed on line $FILE_NO of the config file."
    echo "Continuing to build archive list..."
    echo
fi
FILE_NO=$[ $FILE_NO + 1 ] #Icrease Line/Flie number by one.
```
说明：
- 由于归档配置文件中的记录可能是文件名，也可能是目录名，所以if语句总用`-f`和`-d`选项分别测试是否存在
- or选项`-o`则表示只要一个测试为真，那么整个if语句就成立
- 添加变量FILE_NO可以靠苏用户归档配置文件中哪行中含有不正确或确实的文件或目录

#### 创建逐日归档文件的存放位置
如果要对多个目录进行备份，最好创建一个集中归档仓库目录（注意权限）：
```
[root@redhat8 tool]# mkdir /archive
[root@redhat8 tool]# ls -ld /archive
drwxr-xr-x. 2 root root 6 Dec 20 02:19 /archive
[root@redhat8 tool]# mv Files_To_Backup /archive/
```
可以创建一个用户组，为需要在集中归档目录中创建文件的用户授权：
```
[root@redhat8 tool]# groupadd Archivers
[root@redhat8 tool]# chgrp Archivers /archive
[root@redhat8 tool]# ls -ld /archive
drwxr-xr-x. 2 root Archivers 29 Dec 20 02:20 /archive
[root@redhat8 tool]# usermod -aG Archivers huang
[root@redhat8 tool]# chmod 775 /archive
[root@redhat8 tool]# ls -ld /archive
drwxrwxr-x. 2 root Archivers 29 Dec 20 02:20 /archive
```
注意：
- 将用户添加到Archivers组后，如果当前正在使用此用户，需要登出再登入才能使组成员关系生效
- Archivers组中所有用户都可以再归档目录中添加和删除文件，为了避免组用户删除他人的归档文件，最好把目录的粘滞位加上

#### 创建按日归档的脚本
Daily_Archive脚本会自定再指定位置创建一个归档，使用当前日期来唯一标识该文件：
```sh
DATE=$(date +%y%m%d)
# Set Archive File Name
FILE=archive$DATE.tar.gz
# Set Configuration and Destination File
CONFIG_FILE=/archive/Flies_To_Backup
DESTINATION=/archive/$FILE
```
说明：
- DESTINATION变量会将归档文件的全路径名加上去
- CONFIG_FILE变量指向含有待归档目录信息的归档配置文件

将前面内容结合在一起，Daily_Archive脚本如下
```sh
#!/bin/bash
# Daily_Archive - Archive designated files & directories
########################################################
# Gather Current Date
DATE=$(date +%y%m%d)
# Set Archive File Name
FILE=archive$DATA.tar.gz
# Set Configuration and Destination File
CONFIG_FILE=/archive/Files_To_Backup
DESTINATION=/archive/$FILE
############# Main Script ##############################
# Check Backup config file exists
if [ -f $CONFIG_FILE ] #Make sure the config file still exists.
then          # if file exists,do nothing but continue on.
    echo
else          # if file doernot exist,issue warning & eit script.
    echo "$CONFIG_FILE,does not exist."
    echo "Backup not completed due to missing Configuration File "
    exit
fi
# Build the names of all the files to backup
FILE_NO=1             #Start on line 1 of Config File.
exec < $CONFIG_FILE   #Redirect Std Input to name of Config File
read FILE_NAME        #Read 1st record
while [ $? -eq 0 ]    #Create list of files to backup
do
    # Make sure the file of directory exists.
    if [ -f $FILE_NAME -o -d $FILE_NAME ]
    then
        # if file exists,add its name to the list.
        FILE_LIST="$FILE_LIST $FILE_NAME"
    else
        # if file doernot exist,issue warning
        echo
        echo "$FILE_NAME,does not exist."
        echo "Obviously,I will not include it in this archive."
        echo "It is listed on line $FILE_NO of the config file."
        echo "Continuing to build archive list..."
        echo
    fi
    FILE_NO=$[ $FILE_NO + 1 ] #Icrease Line/Flie number by one.
    read FILE_NAME            #Read next record
done
#################################################################
#Back up the files and Compress Archive
echo "Staring archive..."
echo
tar -czf $DESTINATION $FILE_LIST 2> /dev/null
echo "Archive completed"
echo "Resulting archive file is: $DESTINATION"
exit
```
#### 运行按日归档的脚本
首先赋予执行权限：
```
[root@redhat8 tool]# ls -l Daily_Archive.sh
-rw-r--r--. 1 root root 1882 Dec 20 04:02 Daily_Archive.sh
[root@redhat8 tool]# chmod u+x Daily_Archive.sh
[root@redhat8 tool]# ls -l Daily_Archive.sh
-rwxr--r--. 1 root root 1882 Dec 20 04:02 Daily_Archive.sh
```
运行脚本进行测试：
```
[root@redhat8 tool]# ./Daily_Archive.sh

/shell/funciton,does not exist.
Obviously,I will not include it in this archive.
It is listed on line 2 of the config file.
Continuing to build archive list...

Staring archive...

Archive completed
Resulting archive file is: /archive/archive.tar.gz
[root@redhat8 tool]# ls -l /archive/archive.tar.gz
-rw-r--r--. 1 root root 4231 Dec 20 04:10 /archive/archive.tar.gz
```
&#8195;&#8195;在脚本中，发现了一个不存在的目录：/shell/funciton，脚本告诉了这个错误的行在配置文件中的行号，然后继续创建列表和归档数据。
#### 创建按小时归档的脚本
&#8195;&#8195;按小时备份文件，如果使用date命令为每个taball文件加入时间戳，文件名会很长，很不方便。可以为归档文件创建一个目录层级，包含了与一年中各个月份对应的目录，将月的序号作为目录名；而每月目录中又包含与当月隔天对应的目录（用天的序号作为目录名）。    
首先，创建新目录/archive/hourly，并设置适当权限：
```
[root@redhat8 tool]# mkdir /archive/hourly
[root@redhat8 tool]# chgrp Archivers /archive/hourly
[root@redhat8 tool]# chmod 775 /archive/hourly
[root@redhat8 tool]# ls -ld /archive/hourly
drwxrwxr-x. 2 root Archivers 6 Dec 20 07:12 /archive/hourly
```
将按小时归档的配置文件Files_To_Backup移动到该目录：
```
[root@redhat8 hourly]# cat Files_To_Backup
/shell/gawk
/shell/function
/python/jinja2
[root@redhat8 hourly]# pwd
/archive/hourly
```
&#8195;&#8195;脚本必须自动创建对应每月和每天的目录，如果目录已经存在，脚本就会报错。可以使用mkdir命令的`-p`命令行选项，此选项允许在单个命令中创建目录和子目录，就算目录已经存在，也不会产生错误消息。    
Hourly_Archive.sh脚本的前半部分如下：
```sh
#!/bin/bash
# Hourly_Archive - Every hour create an archive
#######################################################
# Set Configuration File
CONFIG_FILE=/archive/hourly/Files_To_Backup
# Set Base Archive Destination Location
BASEDEST=/archive/hourly
# Gather Current Day,Month & Time
DAY=$(date +%d)
MONTH=$(date +%m)
TIME=$(date +%H%M)
# Greate Archive Destination Directory
mkdir -p $BASEDEST/$MONTH/$DAY
# Bulid Archive Destination File Name
DESTINATION=$BASEDEST/$MONTH/$DAY/archive$TIME.tar.gz
################### Main Script #######################
[...]
```
Main Script部分和Daily_Archive脚本完全一样。
#### 运行按小时归档的脚本
修改好权限后运行如下所示：
```
[root@redhat8 tool]# chmod u+x Hourly_Archive.sh
[root@redhat8 tool]# ./Hourly_Archive.sh

Staring archive...

Archive completed
Resulting archive file is: /archive/hourly/12/20/archive0828.tar.gz
[root@redhat8 tool]# ls -l /archive/hourly/12/20/
total 5760
-rw-r--r--. 1 root root 5897582 Dec 20 08:28 archive0828.tar.gz
```
&#8195;&#8195;可以将`TIME=$(date +%k%M)`修改成`TIME=$(date +%k0%M)`，加入0后，所有的单数字小时数都会被加入一个前导数字0（在RHEL中好像不行，我直接将%k改成%H）。测试下在/archive/hourly/12/20目录存在情况下是否有问题：
```
[root@redhat8 tool]# ./Hourly_Archive.sh

Staring archive...

Archive completed
Resulting archive file is: /archive/hourly/12/20/archive0829.tar.gz
[root@redhat8 tool]# ls -l /archive/hourly/12/20/
total 11520
-rw-r--r--. 1 root root 5897582 Dec 20 08:28 archive0828.tar.gz
-rw-r--r--. 1 root root 5897582 Dec 20 08:29 archive0829.tar.gz
```
测试没有问题，可以将脚本放到cron表中。
## 管理用户账户
管理账户除了添加、修改和删除账户，还需要考虑安全问题、保留工作的需求以及对账户的精确管理。
### 需要的功能
删除账户时，至少需要四个步骤：
- 获得正确的待删除用户账户名
- 杀死正在系统上运行属于该账户的进程
- 确认系统中属于该账户的所有文件
- 删除该用户账户

#### 获取正确的账户名
