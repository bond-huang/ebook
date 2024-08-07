# GitBook-构建ebook
## TravisCI构建
参考Travis-CI基础操作中构建方法。
## 服务器构建
&#8195;&#8195;之前Travis-CI用了几年都很，最近发现Travis-CI被限制进行构建，提示积分不够，但是查看还有9100积分，无奈部署到自己阿里云服务器上去。

参考博客：[http://suqiankun.com/](http://suqiankun.com/)
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
info: 16 plugins are installed 
info: 12 explicitly listed 
info: loading plugin "advanced-emoji"... OK 
......
info: loading plugin "theme-default"... OK 
info: found 402 pages 
info: found 48 asset files 
init!
finish!
info: >> generation finished with success in 603.6s !
```
耗时十分钟左右，CPU使用率54%左右，内存750M左右。
下面命令启动服务会重新构建一次：
```
[gitbook@centos82 ebook]$ gitbook serve
Live reload server started on port: 35729
Press CTRL+C to quit ...

info: 16 plugins are installed 
info: 13 explicitly listed 
info: loading plugin "advanced-emoji"... OK 
......
info: loading plugin "theme-default"... OK 
info: found 402 pages 
info: found 48 asset files 
init!
finish!
info: >> generation finished with success in 605.7s ! 

Starting server ...
Serving book on http://localhost:4000
```
### Nginx配置
构建文件赋予权限：
```
[root@centos82 ~]# chown -R nginx:nginx /home/gitbook/ebook/_book
[root@centos82 ~]# chmod -R 755 /home/gitbook/ebook/_book
```
如果赋权不行，将Nginx配置文件中用户改为root：
```
user  root;
```
在Nginx配置文件中加入配置：
```ini
    server {
        listen       81;
        server_name  localhost;

        location / {
            root   /home/gitbook/ebook/_book;
            index  index.html;
        }
    }
```
保存配置文件，重启Ningx：
```
[root@centos82 nginx]# systemctl reload nginx
```
## 重新构建
### 更新服务器数据
&#8195;&#8195;服务器上没有配置git，github上更新后，需要删除原来的文件，把文件下载到本地，重新手动导入到服务器重新构建。比较麻烦，偶尔弄下。

首先移出`node_modules`文件夹：
```
[gitbook@centos82 ebook]$ mv node_modules ../
[gitbook@centos82 ebook]$ cd ..
[gitbook@centos82 ~]$ ls -l
total 8
drwxrwxr-x 23 gitbook gitbook 4096 Oct  7 23:56 ebook
drwxrwxr-x 43 gitbook gitbook 4096 Aug 15 14:30 node_modules
```
清空项目文件夹：
```
[gitbook@centos82 ~]$ rm -rf ebook
[gitbook@centos82 ~]$ ls
node_modules
[gitbook@centos82 ~]$ mkdir ebook
[gitbook@centos82 ~]$ ls -l
total 4
drwxrwxr-x  2 gitbook gitbook    6 Oct  7 23:58 ebook
drwxrwxr-x 43 gitbook gitbook 4096 Aug 15 14:30 node_modules
```
下载项目到本地：
```
[gitbook@centos82 ~]$ git clone https://github.com/bond-huang/ebook.git /home/gitbook/ebook
Cloning into '/home/gitbook/ebook'...
remote: Enumerating objects: 8408, done.
remote: Counting objects: 100% (1719/1719), done.
remote: Compressing objects: 100% (603/603), done.
remote: Total 8408 (delta 1099), reused 1635 (delta 1091), pack-reused 6689
Receiving objects: 100% (8408/8408), 11.41 MiB | 8.16 MiB/s, done.
Resolving deltas: 100% (4536/4536), done.
```
生成目录：
```
[gitbook@centos82 ebook]$ pwd
/home/gitbook/ebook
[gitbook@centos82 ebook]$ sh summary_crt.sh
```
移回`node_modules`文件夹：
```
[gitbook@centos82 ~]$ ls
ebook  node_modules
[gitbook@centos82 ~]$ mv node_modules/ ./ebook
[gitbook@centos82 ~]$ ls
ebook
```
构建数据：
```
[gitbook@centos82 ebook]$ gitbook build .
......
info: found 419 pages
info: found 48 asset files
......
info: >> generation finished with success in 718.2s ! 
```
构建完成后web可以访问，不需要重启Nginx。

## 迁移到其它系统
源版本是centos8.2，目标版本是centos7.9。首先安装nodejs：
```
yum install nodejs
```
安装npm：
```
yum install npm
```
安装GitBook-cli：
```
npm install -g gitbook-cli
```
安装很慢，耐心等待：
```
[root@huang ~]# npm list -g
/usr/local/lib
└── gitbook-cli@2.3.2
```
给gitbook用户添加权限，编辑`/etc/sudoers`文件：
```
gitbook ALL=(ALL)       ALL
```
进入gitbook用户然后安装gitbook：
```sh
gitbook -V
```
安装成功，版本和原版本一致：
```
[gitbook@huang ~]$ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```
安装git：

```sh
yum install git
```
可以下载项目重新配置，参考上面文档，可以直接从源系统拷贝：
```sh
scp -r root@121.43.191.157:/home/gitbook/ebook /home/gitbook/ebook
```
新增了内容，重新生产下目录：
```sh
sh summary_crt.sh
```
也重新构建下，如果没有更新没必要：
```sh
gitbook build .
```
安装nginx：
```sh
yum install nginx
```
拷贝nginx配置文件：
```sh
scp  root@121.43.191.157:/etc/nginx/nginx.conf /etc/nginx/
```
拷贝证书：
```sh
scp  root@121.43.191.157:/etc/nginx/big1000* /etc/nginx/
```
拷贝日志文件：
```sh
scp -r root@121.43.191.157:/etc/nginx/logs /etc/nginx/logs
```
启动nginx：
```sh
systemctl start nginx
```
阿里云上防火墙端口打开，443端口，然后访问验证：
```
https://gitbook.big1000.com/
```
## 待补充