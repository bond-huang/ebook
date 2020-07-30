[TOC]
# GitHub-常用操作
目前大部分都是在WEB进行完成，命令行用的少，但是有些时候必须用到。
### 修改或添加目录
最近想修改下目录名字，GitHub的WEB管理界面找了半天没找到，查了下确实没有，需要通过命令行来操作。
建议最好Git上远程仓库和GitHub上保持一致再作修改目录操作，不一致可以可以把远程仓库更新同步到本地。
##### 同步到本地
使用命令`git pull`,作用是从另外个仓库或者本地分支获取内容并合并，格式如下：
```shell
git pull [<options>] [<repository> [<refspec>…​]]
```
首先查看所在的分支:
```shell
$ git branch
* master
```
我就一个master分支，不需要加参数直接输入命令即可：
```shell
$ git pull
Already up to date.
```
##### 修改目录名
同步完成后进行修改，使用命令`git mv`，示例如下
```shell
$ git mv -f 14-IBM_Hybrid_Cloud 07-IBM_Hybrid_Cloud
#然后查看下
$ ls
 01-IBM_Power_System/                 08-Python/         deploy.sh*
 02-IBM_Z_System/                     09-Shell脚本/      license
 03-IBM_Storage_System/               10-Git/            README.md
 04-IBM_Virtualization/               11-常用操作系统/   SUMMARY.md
 05-IBM_Operating_System/             12-虚拟化平台/     summary_crt.sh*
'06-IBM_Database&Middleware&Other'/   13-X86_System/
 07-IBM_Hybrid_Cloud/                 book.json
```
如果目录中有&这种符号，目录名前后需要有引号
##### 记录对repository的更改
使用`git commit`命令：
```shell
$ git commit -m "changed the foldername"
[master 739ab96] changed the foldername
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename {14-IBM_Hybrid_Cloud => 07-IBM_Hybrid_Cloud}/README.md (100%)
```
##### 同步
使用`git push`命令，就一个master分支，直接输入即可：
```shell
$ git push
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 262 bytes | 131.00 KiB/s, done.
Total 2 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:bond-huang/huang.git
   f21bfa4..c51279c  master -> master
```
在WEB界面刷新下，可以看到更改成功了。
上面命令更多使用方法可以参考官方：[https://git-scm.com/docs](https://git-scm.com/docs)
