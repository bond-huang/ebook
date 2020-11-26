# Python运维笔记-基础知识
记录一些基础知识学习笔记。学习教材：《Python Linux系统管理与自动化运维》（赖明星著）
## Python内置小工具
### 下载服务器
&#8195;&#8195;Linux传输文件通常使用ftp等，用命令有时感觉不方便，Python有个内置web服务器，可以作为一个下载服务器，在Python3中，使用示例：
```
[root@redhat8 shell]# ls
basis  for  function  gawk  if-for  input  instance  output  regular  sed  sed_gawk  test
[root@redhat8 shell]# python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
&#8195;&#8195;然后打开浏览器，输入地址`http://192.168.18.131:8000/`既可，可以看到类似FTP下载页面，使用笔记方便。如果目录下存在一个index.html的文件，默认显示该文件内容，如果没有，默认显示当前目录下的文件列表。

在Python2中的使用方法：
```
python -m SimpleHTTPServer
```
### 字符串转换为JSON
&#8195;&#8195;JSON是一种轻量级的数据交换格式，网上可以搜索到在线JSON格式化工具，当然可以在命令行的Python解析器来解析JSOS串。使用示例：
```
[root@redhat8 shell]# echo '{"job":"devops","name":"bond","sex":"male"}'|python3 -m json.t
ool{
    "job": "devops",
    "name": "bond",
    "sex": "male"
}
```
还可以自动对齐和格式化，示例：
```
[root@redhat8 shell]# echo '{"address":{"province":"guangdong","city":"shenzhen"},"name":"
bond","sex":"male"}'|python3 -m json.tool
{
    "address": {
        "province": "guangdong",
        "city": "shenzhen"
    },
    "name": "bond",
    "sex": "male"
}
```
### 检查第三方库
检查第三方库是否正确安装，只需要尝试import即可，不报错就没问题：
```
[root@redhat8 shell]# python3
Python 3.6.8 (default, Jan 11 2019, 02:17:16) 
[GCC 8.2.1 20180905 (Red Hat 8.2.1-3)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> improt paramiko
```
或者使用`-c`参数快读执行`import`语句:
```
[root@redhat8 python]# python3 -c "import paramiko"
```
## pip高级用法
Python生态主流的包管理工具是pip。
### pip介绍
pip是用来安装和管理Python包的工具，手动安装：
```
$sudo apt-get indstall python-pip
```
我系统中配置了软件仓库，安装时候提示已经安装了：
```
[root@redhat8 python]# yum install python3-pip.noarch
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription
-manager to register.redhat8_app                                               0.0  B/s |   0  B     00:00    
redhat8_os                                                0.0  B/s |   0  B     00:00    
Failed to synchronize cache for repo 'redhat8_app', ignoring this repo.
Failed to synchronize cache for repo 'redhat8_os', ignoring this repo.
Package python3-pip-9.0.3-13.el8.noarch is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```
但是输入`pip`命令无效，查找以下即可：
```
[root@redhat8 python]# whereis pip
pip: /usr/bin/pip3.6
[root@redhat8 python]# pip3.6 install -U pip
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip3.
6 install --user` instead.
```
有些软件没有上传到pip网站，可以使用源码安装方式,示例：
```
$ git clone https://github.bom/paramiko/paramiko.git
$ cd paramiko
$ python setup.py install
```
### pip常用命令
常用子命令如下：

命令|说明
:---|:---
install|安装软件包
download|下载软件包
uninstall|卸载安装包
freeze|按照requirements的格式输出按照包
list|列出当前系统中的安装包
show|查看安装包信息
check|检查安装包的依赖是否完整
search|查找安装包
wheel|打包软件到wheel格式
hash|计算按照包的hash值
completion|生成命令补全配置
help|获取pip和子命令的帮助信息

导出已安装的软件包列表到requirements文件，并从requirements文件按照示例：
```
$ pip freeze > requirements.tex
$ pip install -r requirements.tex
```
### pip加速安装技巧
使用豆瓣的pypi镜像源,示例安装flask：
```
$ pip install -i https://pypi.douban.com/simple/ flask
$ pip install -i https://mirrors.aliyun.com/pypi/simple/ flask
```
或者将镜像源写入到配置文件中：
```
[root@redhat8 /]# cd ~
[root@redhat8 ~]# mkdir .pip
[root@redhat8 ~]# cd .pip
[root@redhat8 .pip]# touch pip.conf
[root@redhat8 .pip]# vim pip.conf
[root@redhat8 .pip]# pwd
/root/.pip
[root@redhat8 .pip]# cat pip.conf
[global]
timeout = 20
index-url=https://mirrors.aliyun.com/pypi/simple/  
extra-index-url=https://pypi.douban.com/simple/
[install]
trusted-host=
    mirrors.aliyun.com
    pypi.douban.con
```
也可以下载到本地，然后再安装：
```
$ pip install --download=`pwd` -r requirements.txt
$ pip install --no-index -f file://`pwd` -r requirements.txt
```
pip能自动处理软件的依赖问题，里面下载flask时候，依赖包都会被下载到本地：
```
$ pip install --download=`pwd` flask
```
## Python编辑器
### vim插件
vim用的少，《Python Linux系统管理与自动化运维》书中介绍了三个，具体见书中：
- 一键执行插件:写完测试后不用退出vim，立即执行就能看到结果
- 代码补全插件snipmate：按tab键补全，方便快捷
- 语法检查插件Syntastic：提示哪些代码存在语法错误，哪些代码不符合编码规范，并给出具体提示信息
- 编程提示插件jedi-vim：基于jedi的自动补全插件

### PyCharm
是目前最流行的Python IDE,目前没使用，具体介绍见《Python Linux系统管理与自动化运维》。
## Python编程辅助工具
《Python Linux系统管理与自动化运维》书中介绍了不少，之前有朋友推荐过jupyter，虽然没用过，感觉很屌。
### jupyter的使用
jupyter是一种新兴的交互式数据分析与记录工具。     
官方网站：[https://jupyter.org/](https://jupyter.org/)

安装jupyter：
```
[root@redhat8 python]# pip3.6 install jupyter
[root@redhat8 python]# pip3.6 install -i https://pypi.douban.com/simple/ jupyter
[root@redhat8 python]# pip3.6 install -i https://mirrors.aliyun.com/pypi/simple/ jupyter
```
虚拟机RedHat怎么也安装不成功，在windows下安装成功：
```
C:\Users\QianHuang>pip install -i https://mirrors.aliyun.com/pypi/simple/ jupyter
Looking in indexes: https://mirrors.aliyun.com/pypi/simple/
Collecting jupyter
  Downloading https://mirrors.aliyun.com/pypi/packages/83/df/0f5dd132200728a86190397e1ea87cd76244e42d39ec5e88efd25b2abd7e/jupyter-1.0.0-py2.py3-none-any.whl (2.7 kB)
Collecting jupyter-console
......
WARNING: You are using pip version 20.2.3; however, version 20.2.4 is available.
You should consider upgrading via the 'c:\users\qianhuang\appdata\local\programs\python\python38\python.exe -m pip install --upgrade pip' command.
```
设置浏览器进行外部访问：
```
C:\Users\QianHuang>jupyter notebook --no-browser
[I 00:16:09.394 NotebookApp] Writing notebook server cookie secret to C:\Users\QianHuang\AppData\Roaming\jupyter\runtime\notebook_cookie_secret
[I 00:16:10.298 NotebookApp] Serving notebooks from local directory: C:\Users\QianHuang
[I 00:16:10.298 NotebookApp] Jupyter Notebook 6.1.5 is running at:
[I 00:16:10.300 NotebookApp] http://localhost:8888/?token=6ba5db71f72213c9941d66f981330bd7e21ad70ff73d395f
[I 00:16:10.301 NotebookApp]  or http://127.0.0.1:8888/?token=6ba5db71f72213c9941d66f981330bd7e21ad70ff73d395f
[I 00:16:10.302 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 00:16:10.327 NotebookApp]
    To access the notebook, open this file in a browser:
        file:///C:/Users/QianHuang/AppData/Roaming/jupyter/runtime/nbserver-18428-open.html
    Or copy and paste one of these URLs:
        http://localhost:8888/?token=6ba5db71f72213c9941d66f981330bd7e21ad70ff73d395f
     or http://127.0.0.1:8888/?token=6ba5db71f72213c9941d66f981330bd7e21ad70ff73d395f
```
根据提示链接，打开浏览器即可访问，可以加上`--ip`来指定地址，默认是localhost：
```
C:\Users\QianHuang>jupyter notebook --no-browser --ip=0.0.0.0
```
## Python调试器

## Python代码检查规范
