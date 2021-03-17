# Docker-部署Flask应用
以写的一个导航工具为例，代码托管在GitHub：[https://github.com/bond-huang/navigator](https://github.com/bond-huang/navigator)
配置参考链接：
- [Docker runoob网站教程](https://www.runoob.com/docker/centos-docker-install.html)
- [uWSGI文档](https://uwsgi-docs.readthedocs.io/en/latest/)
- [Flask部署方式](https://dormousehole.readthedocs.io/en/latest/deploying/index.html)
- [ngix官方安装文档](https://nginx.org/en/linux_packages.html#instructions)
- [Python uWSGI 安装配置](https://www.runoob.com/python3/python-uwsgi.html)

## 安装CentOS
安装CentOS7：
```
[root@VM-0-6-centos ~]# docker pull centos:7
7: Pulling from library/centos
2d473b07cdd5: Pull complete 
Digest: sha256:0f4ec88e21daf75124b8a9e5ca03c37a5e937e0e108a255d890492430789b60e
Status: Downloaded newer image for centos:7
docker.io/library/centos:7
```
查看镜像：
```
[root@VM-0-6-centos ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   9 days ago     13.3kB
centos        centos7   8652b9f0cb4c   4 months ago   204MB
```
运行容器并进入CentOS容器：
```
[root@VM-0-6-centos ~]# docker run -itd --name navigator centos:centos7
554b9bc044751e94f5e9f9802bdfc49f05ff55d0bc0f05913b0f27fd3e0be512
[root@VM-0-6-centos ~]# docker ps
CONTAINER ID   IMAGE            COMMAND       CREATED         STATUS         PORTS    NAMES
554b9bc04475   centos:centos7   "/bin/bash"   2 minutes ago   Up 2 minutes        navigator
[root@VM-0-6-centos ~]# docker exec -it navigator /bin/bash
[root@554b9bc04475 /]# 
```
说明：
- `-i`: 交互式操作
- `-t`: 终端
- `/bin/bash`:shell类型

## 拷贝项目文件
首先将项目文件拷贝到操作系统中：
```
[root@VM-0-6-centos navigator]# ls
instance  LICENSE  MANIFEST.in  nav  README.md  setup.cfg  setup.py  tests
```
文件拷贝到容器中：
```
# docker cp ./navigator/ 554b9bc04475:/navigator
[root@VM-0-6-centos ~]# docker exec -it navigator /bin/bash
[root@554b9bc04475 navigator]# ls
LICENSE  MANIFEST.in  README.md  instance  nav  setup.cfg  setup.py  tests
```
## 安装基础软件
### 安装python3
安装示例：
```
[root@554b9bc04475 /]# yum install python3
...
Running transaction
  Installing : libtirpc-0.2.4-0.16.el7.x86_64                                         1/5 
  Installing : python3-setuptools-39.2.0-10.el7.noarch                                2/5 
  Installing : python3-pip-9.0.3-8.el7.noarch                                         3/5 
  Installing : python3-3.6.8-18.el7.x86_64                                            4/5 
  Installing : python3-libs-3.6.8-18.el7.x86_64                                       5/5
...
[root@554b9bc04475 /]# python3
Python 3.6.8 (default, Nov 16 2020, 16:55:22) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-44)] on linux
Type "help", "copyright", "credits" or "license" for more information.
```
pip3那些都自动安装了。
### 安装项目需求包
安装Flask（不推荐用root）：
```
[root@554b9bc04475 /]# pip3 install Flask
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip3 
install --user` instead.Collecting Flask
...
```
Flask需要的例如jinja2和click都安装了。
### 安装waitress
安装waitress：
```
[root@554b9bc04475 /]# pip3 install waitress
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip3 
install --user` instead.Collecting waitress
  Downloading https://files.pythonhosted.org/packages/a8/cf/a9e9590023684dbf4e7861e261b0cf
d6498a62396c748e661577ca720a29/waitress-2.0.0-py3-none-any.whl (56kB)    100% |################################| 61kB 486kB/s 
Installing collected packages: waitress
Successfully installed waitress-2.0.0
```
### 安装pytest和coverage
安装pytest和coverage：
```
[root@554b9bc04475 /]# pip3 install pytest coverage
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip3 
install --user` instead.
...
```
安装环境：
```
[root@554b9bc04475 navigator]# pip3 install -q -e .
```
### 运行测试
运行测试：
```
[root@554b9bc04475 navigator]# python3 setup.py test
...
E       AssertionError: assert 'Initialized' in ''
E        +  where '' = <Result RuntimeError('Click will abort further execution because Py
thon 3 was configured to use ASCII as encoding for ...IE.utf8, en_IN.utf8, en_NG.utf8, en_NZ.utf8, en_PH.utf8, en_SG.utf8, en_US.utf8, en_ZA.utf8, en_ZM.utf8, en_ZW.utf8',)>.output
...
```
设置下环境变量：
```
[root@554b9bc04475 navigator]# export LC_ALL=en_US.utf-8
[root@554b9bc04475 navigator]# export LANG=en_US.utf-8
```
参考链接：[Click will abort further execution because Python 3 was configured to use ASCII as encoding for the environment](https://stackoverflow.com/questions/36651680/click-will-abort-further-execution-because-python-3-was-configured-to-use-ascii)

再次运行：
```
[root@554b9bc04475 navigator]# python3 setup.py test
running test
================================== test session starts ===================================
platform linux -- Python 3.6.8, pytest-6.2.2, py-1.10.0, pluggy-0.13.1
rootdir: /navigator, configfile: setup.cfg, testpaths: tests
collected 8 items                                                                        
tests/test_db.py ..                                                                [ 25%]
tests/test_factory.py ..                                                           [ 50%]
tests/test_navigator.py ....                                                       [100%]

=================================== 8 passed in 0.34s ====================================
```
使用pytest和coverage：
```
[root@554b9bc04475 navigator]# coverage run -m pytest
================================== test session starts ===================================
platform linux -- Python 3.6.8, pytest-6.2.2, py-1.10.0, pluggy-0.13.1
rootdir: /navigator, configfile: setup.cfg, testpaths: tests
collected 8 items                                                                        

tests/test_db.py ..                                                                [ 25%]
tests/test_factory.py ..                                                           [ 50%]
tests/test_navigator.py ....                                                       [100%]

=================================== 8 passed in 0.31s ====================================
[root@554b9bc04475 navigator]# coverage report
Name                        Stmts   Miss Branch BrPart  Cover
-------------------------------------------------------------
nav/__init__.py                20      0      2      0   100%
nav/db.py                      24      0      4      0   100%
nav/navigation.py              72     12     28     12    76%
nav/templates/footer.html       0      0      0      0   100%
-------------------------------------------------------------
TOTAL                         116     12     34     12    84%
```
### 运行应用
通过nohup后台运行：
```
[root@554b9bc04475 navigator]# nohup waitress-serve --call 'nav:create_app' > nohup.log 2>
&1 &
[1] 175
[root@554b9bc04475 navigator]# ps -ef |grep waitress
root       175    46  0 16:55 pts/1    00:00:00 /usr/bin/python3 /usr/local/bin/waitress-s
```
## 网络端口映射
外部访问容器中可以运行的网络应用，可以通过`-P`或`-p`参数来指定端口映射。

首先创建一个镜像：
```
[root@VM-0-6-centos huang]# docker commit 554b9bc04475 nav
sha256:3bddebade85b20fcd3a86ec3157c71d2f4f1b5a82154a5e7c70d2cd3e86d5066
```
查看镜像：
```
[root@VM-0-6-centos huang]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED              SIZE
nav           latest    3bddebade85b   28 seconds ago       365MB
```
运行镜像：
```
[root@VM-0-6-centos huang]# docker run -itd -p 8080:8080 --name nav nav:latest
[root@VM-0-6-centos huang]# docker ps
CONTAINER ID   IMAGE        COMMAND       CREATED         STATUS         PORTS                    NAMES
cd414072bc2b   nav:latest   "/bin/bash"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->8080/tcp   nav
```
进入容器启动服务：
```
[root@VM-0-6-centos huang]# docker exec -it nav /bin/bash
[root@cd414072bc2b /]# cd navigator
[root@cd414072bc2b navigator]# 
[root@cd414072bc2b navigator]# nohup waitress-serve --call 'nav:create_app' > nohup.log 2>
&1 &
[1] 29
```
然后就可以宿主机器IP加端口访问此项目，如果是外网，加一个域名解析可以用域名访问示例：[http://nav.big1000.com:8080/](http://nav.big1000.com:8080/)

## uWSGI部署应用
### 安装软件
安装uWSGI：
```
[root@cd414072bc2b navigator]# pip3 install uwsgi
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip3 
install --user` instead.Collecting uwsgi
  Downloading https://files.pythonhosted.org/packages/c7/75/45234f7b441c59b1eefd31ba3d1041
a7e3c89602af24488e2a22e11e7259/uWSGI-2.0.19.1.tar.gz (803kB)    100% |################################| 808kB 29kB/s 
Installing collected packages: uwsgi
  Running setup.py install for uwsgi ... error
...
Command "/usr/bin/python3 -u -c "import setuptools, tokenize;__file__='/tmp/pip-build-5uni
whic/uwsgi/setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record /tmp/pip-n1gtdb69-record/install-record.txt --single-version-externally-managed --compile" failed with error code 1 in /tmp/pip-build-5uniwhic/uwsgi/
```
报错解决方法：
```
[root@cd414072bc2b navigator]# yum install gcc python36-devel
...
Package python3-devel-3.6.8-18.el7.x86_64 already installed and latest version
...
Installed:
  gcc.x86_64 0:4.8.5-44.el7                                                               
Dependency Installed:
  cpp.x86_64 0:4.8.5-44.el7                glibc-devel.x86_64 0:2.17-323.el7_9            
  glibc-headers.x86_64 0:2.17-323.el7_9    kernel-headers.x86_64 0:3.10.0-1160.15.2.el7   
  libgomp.x86_64 0:4.8.5-44.el7            libmpc.x86_64 0:1.0.1-3.el7                    
  mpfr.x86_64 0:3.1.1-4.el7               
Dependency Updated:
  glibc.x86_64 0:2.17-323.el7_9            glibc-common.x86_64 0:2.17-323.el7_9           
Complete!
```
参考链接：[Compile failed with error code 1 in /tmp/pip_build_root/uwsgi](https://stackoverflow.com/questions/29640868/compile-failed-with-error-code-1-in-tmp-pip-build-root-uwsgi)

再次安装：
```
[root@cd414072bc2b navigator]# pip3 install uwsgi
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip3 
install --user` instead.Collecting uwsgi
  Using cached https://files.pythonhosted.org/packages/c7/75/45234f7b441c59b1eefd31ba3d104
1a7e3c89602af24488e2a22e11e7259/uWSGI-2.0.19.1.tar.gzInstalling collected packages: uwsgi
  Running setup.py install for uwsgi ... done
Successfully installed uwsgi-2.0.19.1
[root@cd414072bc2b navigator]# pip3 list
uWSGI (2.0.19.1)
``` 
运行：
```
[root@8a5f9985230a navigator]# uwsgi --http 127.0.0.1:5000 --module nav:create_app
...
WSGI app 0 (mountpoint='') ready in 1 seconds on interpreter 0x1cc1100 pid: 132 (default a
pp)
...
```
如果有如下提示表示未成功：
```
unable to load app 0 (mountpoint='') (callable not found or import error)
*** no app loaded. going in full dynamic mode ***
```
## 使用nginx
### nginx安装
`/etc/yum.repos.d/`目录下新建文件nginx.repo：
```
[root@cd414072bc2b yum.repos.d]# touch nginx.repo
```
写入如下内容：
```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```
安装nginx：
```
[root@cd414072bc2b yum.repos.d]# yum install nginx
...
Installed:
  nginx.x86_64 1:1.18.0-2.el7.ngx                                                         
Dependency Installed:
  make.x86_64 1:3.82-24.el7                openssl.x86_64 1:1.0.2k-21.el7_9               
Dependency Updated:
  openssl-libs.x86_64 1:1.0.2k-21.el7_9                                                   
Complete!
```
### 测试nginx
查看位置：
```
[root@cd414072bc2b /]# whereis nginx
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx
```
启动测试：
```
[root@cd414072bc2b nginx]# /usr/sbin/nginx
[root@cd414072bc2b nginx]# ps -ef |grep nginx
root       744     0  0 07:58 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      745   744  0 07:58 ?        00:00:00 nginx: worker process
```
建一个镜像：
```
[root@VM-0-6-centos ~]# docker ps 
CONTAINER ID   IMAGE        COMMAND       CREATED        STATUS        PORTS              
      NAMEScd414072bc2b   nav:latest   "/bin/bash"   14 hours ago   Up 14 hours   0.0.0.0:8080->8080/
tcp   nav[root@VM-0-6-centos ~]# docker commit cd414072bc2b navigator
sha256:9024cf1a2a9e05eff724c1748d4861dc6a171eca95150565e867ae0989ae2ab6
```
启动并进入容器并启动nginx：
```
[root@VM-0-6-centos ~]# docker run -itd -p 80:80 --name navigator navigator:latest
8a5f9985230aeafd1df70c0b1c434da20761f34df0f99a2502c0f32a9990fac3
[root@VM-0-6-centos ~]# docker exec -it navigator /bin/bash
[root@8a5f9985230a /]# /usr/sbin/nginx
```
在浏览器输入宿主机器的IP即可访问nginx的默认页面。

### 配置nginx
配置文件位置：
```
[root@8a5f9985230a conf.d]# pwd
/etc/nginx/conf.d
[root@8a5f9985230a conf.d]# ls
default.conf
```
默认配置文件：
```
[root@8a5f9985230a conf.d]# cat default.conf
server {
    listen       80;
    server_name  localhost;
    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```
修改配置：
```
[root@8a5f9985230a conf.d]# cat default.conf
server {
    listen       80;
    server_name  localhost;
    charset utf-8;
    client_max_body_size 80M;
    #access_log  /var/log/nginx/host.access.log  main;
    location / {
	    include uwsgi_params;
        uwsgi_pass 0.0.0.0:8080;
    }
```
停止服务：
```
[root@8a5f9985230a conf.d]# /usr/sbin/nginx -s stop
[root@8a5f9985230a conf.d]# ps -ef |grep nginx
root        70    15  0 08:34 pts/1    00:00:00 grep --color=auto nginx
```
启动nginx和uwsgi：
```
[root@8a5f9985230a conf.d]# /usr/sbin/nginx
[root@8a5f9985230a conf.d]# uwsgi --http 0.0.0.0:8080 --module nav:create_app
```
使用url去测试，发现是空的：
```
[root@8a5f9985230a /]# curl localhost:8080/gump
curl: (52) Empty reply from server
```
uwsgi报错：
```
TypeError: create_app() takes from 0 to 1 positional arguments but 2 were given
[pid: 210|app: 0|req: 1/1] 127.0.0.1 () {28 vars in 302 bytes} [Tue Mar 16 14:37:02 2021] GET /gump => generated 0 bytes in 0 msecs (HTTP/1.1 500) 0 headers in 0 bytes (0 switches on core 0)
```
写一个启动文件：
```py
import os
from nav import create_app
myapp = create_app()
if __name__ == "__main__":
    myapp.run(host='0.0.0.0', port=8080)
```
运行测试可以看到请求正常：
```
[root@8a5f9985230a navigator]# python3 start.py
 * Serving Flask app "nav" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
127.0.0.1 - - [16/Mar/2021 16:10:37] "GET /gump HTTP/1.1" 200 -
```
再次运行：
```
[root@8a5f9985230a navigator]# uwsgi --http 0.0.0.0:8080 --module start:myapp
```
测试成功：
```
[root@8a5f9985230a /]# curl localhost:8080/gump
Life was like a box of chocolates,you never know what you're gonna get.
```
问题解决参考链接：[https://www.pythonanywhere.com/forums/topic/8397/](https://www.pythonanywhere.com/forums/topic/8397/)

启动nginx打开网页报错：
```
An error occurred.
Sorry, the page you are looking for is currently unavailable.
Please try again later.
If you are the system administrator of this resource then you should check the error log for details.
Faithfully yours, nginx.
```
查看宿主机开启端口：
```
[root@VM-0-6-centos ~]# netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5398/docker-proxy   
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      14735/docker-proxy  
tcp        0      0 0.0.0.0:38002           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1387/sshd           
tcp6       0      0 :::44916                :::*                    LISTEN      -   
```
#### 重头开始
&#8195;&#8195;看了一些网上资料，内容比较杂，各不相同，照着试还是各种失败，最后回到runoob教程，写的很简单，但是运行成功了，链接：[Python uWSGI 安装配置](https://www.runoob.com/python3/python-uwsgi.html)

start.py内容：
```py
import os
from nav import create_app
myapp = create_app()
if __name__ == "__main__":
    myapp.run()
```
nginx配置：
```
server {
    listen       80;
    server_name  localhost;

    #charset utf-8;
    client_max_body_size 80M;
    access_log  /navigator/logs/access.log;
    error_log /navigator/logs/error.log;

    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:3031;
        }
```
uwsgi启动命令：
```
uwsgi --socket 127.0.0.1:3031 --wsgi-file start.py --callable myapp --processes 4 --threads 2 --stats 127.0.0.1:9191
```
容器中测试：
```
[root@8a5f9985230a conf.d]# curl 127.0.0.1:80/gump
Life was like a box of chocolates,you never know what you're gonna get.
```
宿主机测试：
```
[root@VM-0-6-centos ~]# curl 127.0.0.1:80/gump
Life was like a box of chocolates,you never know what you're gonna get.
```
外网访问正常：
```
[pid: 615|app: 0|req: 2/11] 223.73.220.199 () {40 vars in 578 bytes} [Wed Mar 17 16:32:52 
2021] GET /static/nav.js => generated 698 bytes in 2 msecs via sendfile() (HTTP/1.1 200) 7 headers in 291 bytes (0 switches on core 0)
```
保持后台运行，在项目目录下的文件uwsgi.ini:
```ini
[uwsgi]
socket = 127.0.0.1:3031
wsgi-file = start.py
callable = myapp
processes = 4
threads = 2
stats = 127.0.0.1:9191
daemonize = /navigator/logs/server.log
```
启动uwsgi：
```
[root@8a5f9985230a navigator]# uwsgi uwsgi.ini
[uWSGI] getting INI configuration from uwsgi.ini
```
容器中测试：
```
Internal Server Error[root@8a5f9985230a conf.d]# curl 127.0.0.1:80/gump
Life was like a box of chocolates,you never know what you're gonna get.
```
宿主机测试：
```
[root@VM-0-6-centos ~]# curl 127.0.0.1:80/gump
Life was like a box of chocolates,you never know what you're gonna get.
```
## 补充说明
### 注意事项
注意事项：
- 示例中日志文件的文件夹要存在，不然会报错
- 如果使用nginx来把请求发送给uwsgi，使用socket方式（为tcp协议）进行通信
- 之前没有使用nginx示例中，打开浏览器访问uwsgi指定的端口，请求uwsgi的方式为http协议
- 如果设置成socket，没启动nginx直接访问会报错：invalid request block size

### nginx常用命令
常用命令：
```sh
/usr/sbin/nginx
/usr/sbin/nginx -s stop
/usr/sbin/nginx -s quit
/usr/sbin/nginx -s reload
```
