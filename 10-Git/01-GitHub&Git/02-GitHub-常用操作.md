# GitHub-常用操作
目前大部分都是在WEB进行完成，命令行用的少，但是有些时候必须用到。
## 命令行修改
### 修改或添加目录
&#8195;&#8195;最近想修改下目录名字，GitHub的WEB管理界面找了半天没找到，查了下确实没有，需要通过命令行来操作。建议最好Git上远程仓库和GitHub上保持一致再作修改目录操作，不一致可以可以把远程仓库更新同步到本地。
#### 同步到本地
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
#### 修改目录名
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
#### 记录对repository的更改
使用`git commit`命令：
```shell
$ git commit -m "changed the foldername"
[master 739ab96] changed the foldername
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename {14-IBM_Hybrid_Cloud => 07-IBM_Hybrid_Cloud}/README.md (100%)
```
#### 同步
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

## 常用配置
### 设置Custom domain
在仓库根目录创建CNAME文件，写入自己的域名：
```
big1000.com
```
然后配点击`Setting`,在Custom domain选项中写入自己的地址，然后点击`save`,可以看到如下提示：
```
Your site is ready to be published at http://big1000.com/. 
```
然后登录到自己的域名管理网站（我是阿里云万网），添加一条解析记录：
- 记录类型选择：CNAME
- 主机记录默认空着（创建两条解析记录，第二条写入www）
- 解析线路默认
- 在记录值里面输入对应的github地址：bond-huang.github.io

设置完成后，在浏览器输入链接可以访问：[https://big1000.com/](https://big1000.com/)

配置参考链接：[https://blankj.com/gitbook/gitbook/](https://blankj.com/gitbook/gitbook/)        
配置参考链接：[https://www.cnblogs.com/liangmingshen/p/9561994.html](https://www.cnblogs.com/liangmingshen/p/9561994.html)

### 修改git用户名及提交邮箱
刚开始本地git使用的用户名和邮箱和远程不一致，想改成一致，输入如下命令即可：
```
$ git config  --global user.name <name>
$ git config  --global user.email <emali>
```
说明：不加`--global`就是更改当前的project的信息。

修改后查看配置：
```
$ git config user.name
<name>
$ git config user.email
<emali>
```
或者直接改文件：`vi ~/.gitconfig`。

参考链接：[https://www.cnblogs.com/shenxiaolin/p/7896489.html](https://www.cnblogs.com/shenxiaolin/p/7896489.html)

## 其他常用操作
### Markdown字数统计
&#8195;&#8195;打开Git Bash命令行，cd到项目所在的目录，首先查找所有md文件，遍历后使用`wc`命令进行字数统计。示例统计字符数：
```sh
$ find . -name '*.md' -exec cat {} \;|wc -m
2185910
```
统计词数（汉字一句话算一个词了，以空格等符号做分隔符）：
```sh
$ find . -name '*.md' -exec cat {} \;|wc -w
169617
```
统计行数：
```sh
$ find . -name '*.md' -exec cat {} \;|wc -l
63021
```
## 待补充