# Flask-基础
学习Flask基础笔记，边学习边实践。
## 安装
当安装 Flask 时，以下配套软件会被自动安装。
- Werkzeug:用于实现WSGI,应用和服务之间的标准 Python 接口
- Jinja:用于渲染页面的模板语言
- MarkupSafe与Jinja 共用，在渲染页面时用于避免不可信的输入，防止注入攻击
- ItsDangerous:保证数据完整性的安全标志数据，用于保护Flask的session cookie
- Click:是一个命令行应用的框架。用于提供flask命令，并允许添加自定义管理命令

### 虚拟环境
本人采用Windows环境
#### Windows下安装
Python2需要首先安装virtualenv（Windows下需要先安装pip）：
```
$ pip install virtualenv
Collecting virtualenv
  Downloading virtualenv-20.0.31-py2.py3-none-any.whl (4.9 MB)
     |████████████████████████████████| 4.9 MB 29 kB/s
Collecting six<2,>=1.9.0
  Downloading six-1.15.0-py2.py3-none-any.whl (10 kB)
Collecting appdirs<2,>=1.4.3
  Downloading appdirs-1.4.4-py2.py3-none-any.whl (9.6 kB)
Collecting filelock<4,>=3.0.0
  Downloading filelock-3.0.12-py3-none-any.whl (7.6 kB)
Collecting distlib<1,>=0.3.1
  Downloading distlib-0.3.1-py2.py3-none-any.whl (335 kB)
     |████████████████████████████████| 335 kB 38 kB/s
Installing collected packages: six, appdirs, filelock, distlib, virtualenv
Successfully installed appdirs-1.4.4 distlib-0.3.1 filelock-3.0.12 six-1.15.0 virtualenv-20.0.31
```
Python3就从这里开始创建一个虚拟环境：
```
$ mkdir osmproject
$ cd osmproject
$ pwd
/d/osmproject
$ python -m venv venv
```
打开CMD激活环境：
```
D:\osmproject>venv\Scripts\activate
(venv) D:\osmproject>
```
在git bash中：
```
$ ls
venv/  venvScriptsactivate

```
安装Flask:
```
$ pip install Flask
Collecting Flask
  Using cached Flask-1.1.2-py2.py3-none-any.whl (94 kB)
Collecting Werkzeug>=0.15
  Using cached Werkzeug-1.0.1-py2.py3-none-any.whl (298 kB)
Collecting itsdangerous>=0.24
  Using cached itsdangerous-1.1.0-py2.py3-none-any.whl (16 kB)
Collecting click>=5.1
  Using cached click-7.1.2-py2.py3-none-any.whl (82 kB)
Collecting Jinja2>=2.10.1
  Using cached Jinja2-2.11.2-py2.py3-none-any.whl (125 kB)
Collecting MarkupSafe>=0.23
  Using cached MarkupSafe-1.1.1-cp38-cp38-win_amd64.whl (16 kB)
Installing collected packages: Werkzeug, itsdangerous, click, MarkupSafe, Jinja2, Flask
Successfully installed Flask-1.1.2 Jinja2-2.11.2 MarkupSafe-1.1.1 Werkzeug-1.0.1 click-7.1.2 itsdangerous-1.1.0
```
克隆仓库到本地：
```
$ git clone git@github.com:bond-huang/OS-Mangement.git
Cloning into 'OS-Mangement'...
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), 4.52 KiB | 578.00 KiB/s, done.
```
创建test.py写入如下python代码：
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def Forrest_Gump():
    return 'Life was like a box of chocolates, you never know what you\'re gonna get.'
```
可以使用`flask run`命令或者`python -m flask run`运行这个应用。在运行应用之前，需要在终端里导出FLASK_APP环境变量:
```
$ export FLASK_APP=test.py
$ python -m flask run
 * Serving Flask app "test.py"
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 ```
 打开浏览器输入地址即可看到相应的网页内容。
