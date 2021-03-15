# TravisCI-部署Python项目
以个人写的navigator为例，代码托管在GitHub，参考官方文档一步一步进行。  
相关链接：   
- Navigator GitHub：[https://github.com/bond-huang/navigator](https://github.com/bond-huang/navigator)
- 官方参考文档：[https://docs.travis-ci.com/user/languages/python/](https://docs.travis-ci.com/user/languages/python/)
- 配置参考链接：[https://github.com/travis-ci-examples/python-example](https://github.com/travis-ci-examples/python-example)

## 基础配置
&#8195;&#8195;首先在项目根目录下创建`.travis.yml`文件，其它配置比如获取GitHub Access Token，注册配置TravisCI，参考前面做的笔记：[TravisCI-基础操作](https://ebook.big1000.com/10-Git/03-Travis-CI/01-TravisCI-%E5%9F%BA%E7%A1%80%E6%93%8D%E4%BD%9C.html)
## 部署Python项目
### 指定python版本
在.travis.yml文件中写入：
```yaml
language: python
python:
  - "3.8"
```
#### Travis CI使用隔离的virtualenvs
&#8195;&#8195;CI环境为每个Python版本使用单独的virtualenv实例。只要`language: python`在`.travis.yml`中指定，就可以在virtualenv内部运行。如果需要安装Python软件包，通过pip而不是apt:

### 依赖管理
默认情况下，Travis CI用于pip管理Python依赖关系。如果项目目录下右requirements.txt文件，Travis CI运行`pip install -r requirements.txt`在install构建阶段：
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install -r requirements.txt
```
此次项目没什么依赖，直接写入：
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install Flask
```
### 指定分支
可以指定分支提交时才运行：
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install Flask
branches:
  only:
    - master
```
### 设置脚本
Python项目需要script在其项目中提供密钥，`.travis.yml`以指定用于运行测试的命令。

例如，如果项目使用pytest：
```yaml
# command to run tests
script: pytest
```
我使用pytest+coverage
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install Flask
  - pip install pytest coverage
branches:
  only:
    - main
script: 
  - coverage run -m pytest
  - coverage report
```
在项目中还使用了waitress：
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install Flask
  - pip install waitress
  - pip install pytest coverage
branches:
  only:
    - main
env:
 global:
    - GH_REF: github.com/bond-huang/navigator.git
script: 
  - coverage run -m pytest
  - coverage report
```
### 环境变量
环境变量可以像加GitHub Access Token一样在Travis CI配置页面添加，可以在此添加
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install Flask
  - pip install waitress
  - pip install pytest coverage
branches:
  only:
    - main
env:
 global:
    - GH_REF: github.com/bond-huang/navigator.git
script: 
  - coverage run -m pytest
  - coverage report
```
测试构建结果：
```sh
Setting environment variables from repository settings
$ export GH_TOKEN=[secure]
$ export M_NAME=[secure]
$ export M_EMAIL=[secure]           0.01s
$ source ~/virtualenv/python3.8/bin/activate
$ python --version
Python 3.8.7
$ pip --version
pip 20.3.3 from /home/travis/virtualenv/python3.8.7/lib/python3.8/site-packages/pip (python 3.8)
install.1                           1.45s
$ pip install Flask
install.2                           0.71s
$ pip install waitress
install.3                           1.24s
$ pip install pytest coverage       0.51s
$ coverage run -m pytest
============================= test session starts ==============================
platform linux -- Python 3.8.7, pytest-6.2.1, py-1.10.0, pluggy-0.13.1
rootdir: /home/travis/build/bond-[secure]/navigator, configfile: setup.cfg, testpaths: tests
collected 8 items
tests/test_db.py ..                                                      [ 25%]
tests/test_factory.py ..                                                 [ 50%]
tests/test_navigator.py ....                                             [100%]
============================== 8 passed in 0.20s ===============================
The command "coverage run -m pytest" exited with 0.
0.12s$ coverage report
Name                        Stmts   Miss Branch BrPart  Cover
-------------------------------------------------------------
nav/__init__.py                21      0      2      0   100%
nav/db.py                      25      0      4      0   100%
nav/navigation.py              76     12     28     12    77%
nav/templates/footer.html       0      0      0      0   100%
-------------------------------------------------------------
TOTAL                         122     12     34     12    85%
The command "coverage report" exited with 0.
Done. Your build exited with 0.
```
pytest和coverage好像自带了，最终我使用配置文件：
```yaml
language: python
python:
  - "3.8"
# command to install dependencies
install:
  - pip install Flask
  - pip install waitress
  - pip install -q -e .
branches:
  only:
    - main
env:
 global:
    - GH_REF: github.com/bond-huang/navigator.git
script:
  - python setup.py test
```
### gh-pages分支
也可在Travis CI以新建一个gh-pages分支：
```yaml
after_script:
  - git config user.name "${M_NAME}"
  - git config user.email "${M_EMAIL}"
  - git config --global core.quotepath false
  - git init
  - git checkout --orphan gh-pages
  - git status
  - sleep 5
  - git add .
  - git commit -m "Update gh-pages"
  - git push --force --quiet "https://${GH_TOKEN}@${GH_REF}" gh-pages:gh-pages
```
## 待补充
