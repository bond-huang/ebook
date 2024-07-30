# Navigator-部署使用
## Linux系统Nginx部署
### 环境准备
系统采用CentOS 8.2版本，各种软件如下：
- Python：Python 3.6.8
- Flask：Flask 0.12.2
- Jinja2：2.10.1
- sqlite3：3.26.0
- uWSGI：2.0.22

安装Flask(如果使用虚拟环境不安装也行)：
```
[root@centos82 ~]# yum install python3-flask
Installed:
  python3-babel-2.5.1-7.el8.noarch           python3-click-6.7-8.el8.noarch          python3-flask-1:0.12.2-4.el8.noarch
  python3-itsdangerous-0.24-14.el8.noarch    python3-jinja2-2.10.1-3.el8.noarch      python3-markupsafe-0.23-19.el8.x86_64
  python3-pytz-2017.2-9.el8.noarch           python3-werkzeug-0.12.2-4.el8.noarch
[root@centos82 ~]# flask --version
Flask 0.12.2
```
安装PIP：
```
[root@centos82 ~]# wget  https://bootstrap.pypa.io/pip/3.6/get-pip.py
[root@centos82 ~]# python3 get-pip.py
[root@centos82 ~]# pip -V
pip 21.3.1 from /usr/local/lib/python3.6/site-packages/pip (python 3.6)
```
安装uWSGI：
```
[root@centos82 ~]# pip install uwsgi
......
Successfully built uwsgi
Installing collected packages: uwsgi
Successfully installed uwsgi-2.0.22
```
安装Nginx：
```
[root@centos82 ~]# yum install nginx
```
### 部署步骤
#### 从GitHub下载项目
使用git将项目拷贝到系统：
```
[navusr@centos82 ~]$ git clone https://github.com/bond-huang/navigator.git ~/navigator
Cloning into '/home/navusr/navigator'...
remote: Enumerating objects: 263, done.
remote: Counting objects: 100% (49/49), done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 263 (delta 23), reused 0 (delta 0), pack-reused 214
Receiving objects: 100% (263/263), 152.64 KiB | 632.00 KiB/s, done.
Resolving deltas: 100% (130/130), done.
```
创建并激活Pyhon虚拟环境：
```
[navusr@centos82 navigator]$ python3 -m venv venv
[navusr@centos82 navigator]$ source venv/bin/activate
(venv) [navusr@centos82 navigator]$
```
如果需要取消激活命令如下：
```
^C(venv) [navusr@centos82 navigator]deactivate
[navusr@centos82 navigator]$
```
安装Flask：
```
(venv) [navusr@centos82 navigator]$ pip install flask
```
#### uwsgi配置
创建uwsgi配置文件：
```ini
[uwsgi]
socket = 127.0.0.1:8080
wsgi-file = start.py
callable = myapp
processes = 4
threads = 2
stats = 127.0.0.1:9191
daemonize = /home/navusr/navigator/logs/server.log
```
#### Nginx配置
Nginx配置文件中的server项写入内容如下：
```ini
server {
    listen       80;
    server_name  localhost;

    #charset utf-8;
    client_max_body_size 80M;
    access_log  /etc/nginx/logs/access.log;
    error_log /etc/nginx/logs/error.log;

    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8080;
        }
}
```
#### 启动应用
添加一个启动文件，名为`start.py`，内容如下：
```python
import os
from nav import create_app
myapp = create_app()
if __name__ == "__main__":
    myapp.run()
```
运行应用：
```
(venv) [navusr@centos82 navigator]$ uwsgi --socket 127.0.0.1:8080 --wsgi-file start.py --callable myapp --processes 4 --threads 2 --stats 127.0.0.1:9191
*** Starting uWSGI 2.0.22 (64bit) on [Tue Aug 15 00:14:06 2023] ***
compiled with version: 8.5.0 20210514 (Red Hat 8.5.0-4) on 14 August 2023 08:55:07
os: Linux-4.18.0-193.14.2.el8_2.x86_64 #1 SMP Sun Jul 26 03:54:29 UTC 2020
nodename: centos82
machine: x86_64
clock source: unix
detected number of CPU cores: 2
current working directory: /home/navusr/navigator
detected binary path: /usr/local/bin/uwsgi
!!! no internal routing support, rebuild with pcre support !!!
your processes number limit is 7185
your memory page size is 4096 bytes
detected max file descriptor number: 65535
lock engine: pthread robust mutexes
thunder lock: disabled (you can enable it with --thunder-lock)
uwsgi socket 0 bound to TCP address 127.0.0.1:8080 fd 5
Python version: 3.6.8 (default, Apr 16 2020, 01:36:27)  [GCC 8.3.1 20191121 (Red Hat 8.3.1-5)]
Python main interpreter initialized at 0x1b9bf30
python threads support enabled
your server socket listen backlog is limited to 100 connections
your mercy for graceful operations on workers is 60 seconds
mapped 416880 bytes (407 KB) for 8 cores
*** Operational MODE: preforking+threaded ***
WSGI app 0 (mountpoint='') ready in 0 seconds on interpreter 0x1b9bf30 pid: 1899 (default app)
*** uWSGI is running in multiple interpreter mode ***
spawned uWSGI master process (pid: 1899)
spawned uWSGI worker 1 (pid: 1901, cores: 2)
spawned uWSGI worker 2 (pid: 1902, cores: 2)
spawned uWSGI worker 3 (pid: 1903, cores: 2)
spawned uWSGI worker 4 (pid: 1905, cores: 2)
*** Stats server enabled on 127.0.0.1:9191 fd: 18 ***
```
启动Nginx：
```
[root@centos82 ~]# nginx
[root@centos82 ~]# ps -aux |grep nginx
root      401413  0.0  0.1 117800  2176 ?        Ss   23:20   0:00 nginx: master process nginx
nginx     401414  0.0  0.4 149304  7956 ?        S    23:20   0:00 nginx: worker process
nginx     401415  0.0  0.4 149304  7956 ?        S    23:20   0:00 nginx: worker process
root      401418  0.0  0.0  12132  1192 pts/1    S+   23:20   0:00 grep --color=auto nginx
```
服务器连接了外网，使用外网IP web登录即可。
#### 后台启动应用
环境启动：
```
[navusr@centos82 navigator]$ pwd
/home/navusr/navigator
[navusr@centos82 navigator]$ source venv/bin/activate
```
从配置文件启动uwsgi：
```
(venv) [navusr@centos82 navigator]$ uwsgi uwsgi.ini
[uWSGI] getting INI configuration from uwsgi.ini
```
启动Nginx：
```
[root@centos82 ~]# nginx
```

## 常见问题
### 虚拟环境启动问题
我从其它系统拷贝过来的目录，执行启动虚拟环境时候报错，示例：
```
[navusr@huang navigator]$ python3 -m venv venv
Error: Command '['/home/navusr/navigator/venv/bin/python3', '-Im', 'ensurepip', '--u
pgrade', '--default-pip']' returned non-zero exit status 1.[navusr@huang navigator]$ ^C
```
使用下面命令即可：
```
[navusr@huang navigator]$ python3 -m venv --without-pip venv
```
参考链接：[Python创建虚拟环境 Error: Command returned non-zero exit status 101.](https://blog.csdn.net/qq_36667170/article/details/124424412)
## 待补充






