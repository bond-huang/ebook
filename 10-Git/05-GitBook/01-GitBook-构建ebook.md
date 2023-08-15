# GitBook-构建ebook
## TravisCI构建
参考Travis-CI基础操作中构建方法。
## 服务器构建
&#8195;&#8195;之前Travis-CI用了几年都很，最近发现Travis-CI被限制进行构建，提示积分不够，但是查看还有9100积分，无奈部署到自己阿里云服务器上去。
### 环境准备
服务器系统是阿里云的CentOS8.2版本，安装node.js：
```
[root@centos82 ~]# yum install nodejs
Installed:
  nodejs-1:10.24.0-1.module_el8.3.0+717+fa496f1d.x86_64
  nodejs-full-i18n-1:10.24.0-1.module_el8.3.0+717+fa496f1d.x86_64
  npm-1:6.14.11-1.10.24.0.1.module_el8.3.0+717+fa496f1d.x86_64
```
安装GitBook-cli：
```
[root@centos82 ~]# npm install -g gitbook-cli
/usr/local/bin/gitbook -> /usr/local/lib/node_modules/gitbook-cli/bin/gitbook.js
+ gitbook-cli@2.3.2
added 578 packages from 672 contributors in 15.678s
```
创建用户gitbook，并添加root权限，修改`/etc/sudoers`文件，添加示例：
```
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
gitbook ALL=(ALL)       ALL
```
安装Gitbook，输入`gitbook -V`自动安装：
```
[gitbook@centos82 ~]$ gitbook -V
CLI version: 2.3.2
Installing GitBook 3.2.3
gitbook@3.2.3 ../../tmp/tmp-9860G0m7y4zDUQ52/node_modules/gitbook
├── escape-html@1.0.3
GitBook version: 3.2.3
[gitbook@centos82 ~]$ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```
### 项目构建
下载项目到本地：
```
[gitbook@centos82 ~]$ git clone https://github.com/bond-huang/ebook.git /home/gitbook/ebook
Cloning into '/home/gitbook/ebook'...
remote: Enumerating objects: 8107, done.
remote: Counting objects: 100% (1418/1418), done.
remote: Compressing objects: 100% (400/400), done.
remote: Total 8107 (delta 997), reused 1408 (delta 993), pack-reused 6689
Receiving objects: 100% (8107/8107), 11.28 MiB | 12.74 MiB/s, done.
Resolving deltas: 100% (4434/4434), done.
```
查看内容：
```
[gitbook@centos82 ~]$ cd ebook
[gitbook@centos82 ebook]$ ls
 01-IBM_Power_System                 07-Oracle_Database   13-X86_System            19-开源软件   summary_crt.sh
'02-IBM_Z&LinuxONE'                  08-Python            14-存储设备              book.json     SUMMARY.md
 03-IBM_Storage_System               09-Shell脚本         15-网络交换机            CNAME
 04-IBM_Virtualization               10-Git               16-云平台                deploy.sh
 05-IBM_Operating_System             11-常用操作系统      17-HTML+CSS+JavaScript   license
'06-IBM_Database&Middleware&Other'   12-虚拟化平台        18-简单WEB项目           README.md
[gitbook@centos82 ebook]$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```
运行脚本生成目录：
```
[gitbook@centos82 ebook]$ cat SUMMARY.md

[gitbook@centos82 ebook]$ sh summary_crt.sh
[gitbook@centos82 ebook]$ cat SUMMARY.md
# Summary

* [Introduction](README.md)
* [SUMMARY](SUMMARY.md)
```
安装插件，以文件`book.json`为基准：
```
[gitbook@centos82 ebook]$ gitbook install
```
构建书籍：
```
[gitbook@centos82 ebook]$ gitbook build .
[gitbook@centos82 ebook]$ gitbook build .
info: 16 plugins are installed
info: 12 explicitly listed
info: loading plugin "advanced-emoji"... OK
......
info: loading plugin "theme-default"... OK
info: found 399 pages
info: found 47 asset files
init!
```
启动服务：
```
[gitbook@centos82 ebook]$ gitbook serve --port 81
```
### Nginx配置

