# SHC-基础环境准备
边学边做，参考文档：
- Vue CLI官方文档：[https://cli.vuejs.org/zh/](https://cli.vuejs.org/zh/)
- CSDN博客(Python之简)：[Flask-Vue前后端分离](https://blog.csdn.net/qq_1290259791/article/details/81174383)
- node.js官网：[https://nodejs.org/en/](https://nodejs.org/en/)

## 软件安装
### 软件类型及版本
需要的软件及版本：
- Python:Python 3.8.3
- Flask:Flask-1.1.2 
- Flask-Cors:Flask-Cors-3.0.10
- Vue.js:@vue/cli 4.5.12
- Node.js:v14.16.1
- npm:6.14.12
- bootstrap: 

### 构建虚环境
构建虚拟环境：
```
$ mkdir flask-vue-shm
$ cd flask-vue-shm
$ python -m venv venv
$ source venv/Scripts/activate
(venv)
$ ls
venv/
(venv)
```
### 测试flask项目
安装Flask-Cors：
```
$ pip install Flask Flask-Cors
...
Successfully installed Flask-1.1.2 Flask-Cors-3.0.10 Jinja2-2.11.3 MarkupSafe-1.1.1 Six-1.15.0 Werkzeug-1.0.1 click-7.1.2 itsdangerous-1.1.0
```
创建测试flask项目，在根目录下创建app.py文件并写入如下内容：
```py
from flask import Flask,jsonify
from flask_cors import CORS
app = Flask(__name__)

CORS(app)
@app.route('/gump')
def Forrest_Gump():
    return jsonify('Life was like a box of chocolates, you never know what you\'re gonna get.')
if __name__ == '__main__':
    app.run()
```
运行项目进行测试：
```sh
$ python app.py
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
在浏览器中输入`http://127.0.0.1:5000/gump`可以看到json格式的内容。

### 安装@vue/cli
下载安装`node.js`，版本`node-v14.16.1-x64`,安装完成后查看版本，然后全局安装`@vue/cli`：
```
C:\Users\QianHuang>node -v
v14.16.1
C:\Users\QianHuang>npm -v
6.14.12
C:\Users\QianHuang>npm install -g @vue/cli
C:\Users\QianHuang>vue --version
@vue/cli 4.5.12
```
&#8195;&#8195;发现目前在git bash的虚拟环境中，运行不了node和vue命令，重启虚拟环境不行，重启下git bash，然后进入虚拟环境，就可以运行vue相关命令。

## 新建项目
### UI创建项目
新建项目步骤如下：
- 运行`vue ui`命令打开UI界面`Vue Project Manager`
- 选择`Create`选项并选择`Create a new project hear`
- 在`Details`页面中输入项目名称，`Package manager`选择默认，`Git repository`建议开启
- 在`Presets`页面中我选择了`Default preset(Vue 3 preview)`
- 点击`Create Project`
- 等待几分钟后，创建成功，进入`Project dashboard`页面
- 回到`Vue Project Manager`界面，在项目后面点击`Open in editor`可以打开VS code进行编辑

### 启动项目
运行下面命令启动项目：
```
$ npm run serve
> shm@0.1.0 serve D:\flask-vue-shm\shm
> vue-cli-service serve
 INFO  Starting development server...
98% after emitting CopyPlugin
 DONE  Compiled successfully in 4938ms                          下午8:47:44
  App running at:
  - Local:   http://localhost:8080/
  - Network: http://192.168.1.4:8080/
  Note that the development build is not optimized.
  To create a production build, run npm run build.
```
我使用Vue3，如果运行下面命令会报错：
```
$ npm run dev
npm ERR! missing script: dev
```
打开浏览器可以看到Vue项目默认主页：Welcome to Your Vue.js App
