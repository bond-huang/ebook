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
 说明：
 - 打开浏览器输入地址即可看到相应的网页内容
 - 运行时候最好在项目主目录运行，要不然会报错
 
 ## github托管
### clone仓库
可以根据之前学习的创建一个项目文件夹，我个人是克隆仓库（空的）到本地：
```
$ git clone git@github.com:bond-huang/OS-Management.git
Cloning into 'OS-Management'...
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), 4.52 KiB | 578.00 KiB/s, done.
```
使用`.gitignore`来设置应当忽略的文件:
```
venv/

*.pyc
__pycache__/

instance/

.pytest_cache/
.coverage
htmlcov/

dist/
build/
*.egg-info/
```
### 同步到github
首先查看状态(列出当前目录所有还未被git管理的文件或被git管理但修改后未提交的文件或目录)：
```
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .gitignore
        osmanagement/
        venvScriptsactivate

nothing added to commit but untracked files present (use "git add" to track)
```
添加一下即可，再次查看会提示有新文件：
```
$ git add .gitignore
warning: LF will be replaced by CRLF in .gitignore.
The file will have its original line endings in your working directory
$ git add osmanagement/
$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .gitignore
        new file:   osmanagement/__init__.py
        new file:   osmanagement/auth.py
        new file:   osmanagement/db.py
        new file:   osmanagement/schema.sql
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        venvScriptsactivate
```
然后提交下,`-m`后面要加参数：
```
$ git commit -m "First commit"
[master 0f5fb28] First commit
 Committer: <name> <email>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 5 files changed, 172 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 osmanagement/__init__.py
 create mode 100644 osmanagement/auth.py
 create mode 100644 osmanagement/db.py
 create mode 100644 osmanagement/schema.sql
```
再次查看状态：
```
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        venvScriptsactivate

nothing added to commit but untracked files present (use "git add" to track)
```
同步到GitHub：
```
$ git remote add origin git@github.com:bond-huang/OS-Management.git
fatal: remote origin already exists.
$ git config --local --list
core.repositoryformatversion=0
core.filemode=false
core.bare=false
core.logallrefupdates=true
core.symlinks=false
core.ignorecase=true
remote.origin.url=git@github.com:bond-huang/OS-Management.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.master.remote=origin
branch.master.merge=refs/heads/master
$ git branch -M master
$ git push -u origin master
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 4 threads
Compressing objects: 100% (8/8), done.
Writing objects: 100% (8/8), 2.46 KiB | 838.00 KiB/s, done.
Total 8 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:bond-huang/OS-Management.git
   f639045..98357c6  master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```
说明：
- 后续提交命令不需要`-u`参数，即`git push origin master`
- 如果就一个master分支，输入`git push`命令即可

### 同步问题
后续在更新本地代码后，想同步到远程，发现如下报错：
```
$ git push origin master
To github.com:bond-huang/OS-Management.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'github.com:bond-huang/OS-Management.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
一般是由于本地和远程不一致造成的，这种不一致表示是远程有的代码本地没有，解决方法：
- 根据提示在使用`push`之前用`git pull`同步到本地，可能会丢失本地的更新
- 使用`-f`参数进行强制更新：`git push -f origin master`，使用要慎重，可能会丢失远程的更新
