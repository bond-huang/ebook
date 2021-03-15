# Docker-部署Flask应用
以写的一个导航工具为例，代码托管在GitHub：[https://github.com/bond-huang/navigator](https://github.com/bond-huang/navigator)
配置参考链接：
- [Docker runoob网站教程](https://www.runoob.com/docker/centos-docker-install.html)

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
&1 &[1] 29
```
然后就可以宿主机器IP加端口访问此项目，如果是外网，加一个域名解析可以用域名访问示例：[http://nav.big1000.com:8080/](http://nav.big1000.com:8080/)

## 待补充
