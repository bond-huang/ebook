# SH-基础环境准备
边学边做，学习参考文档和链接：
- Vue CLI官方文档：[https://cli.vuejs.org/zh/](https://cli.vuejs.org/zh/)
- CSDN博客(Python之简)：[Flask-Vue前后端分离](https://blog.csdn.net/qq_1290259791/article/details/81174383)
- node.js官网：[https://nodejs.org/en/](https://nodejs.org/en/)
- bootcss网站Bootstrap文档：[https://v3.bootcss.com/](https://v3.bootcss.com/)

## 软件安装
使用windows环境下的git bash，代码用VS code编写。
### 软件类型及版本
需要的软件及版本：
- Python:Python 3.8.3
- Flask:Flask-1.1.2 
- Flask-Cors:Flask-Cors-3.0.10
- Vue.js:@vue/cli 4.5.12
- Node.js:v14.16.1
- npm:6.14.12
- Bootstrap:bootstrap@3.3.7
- jQuery:jquery@2.2.4

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

### 添加插件
&#8195;&#8195;默认安装了一些插件。在`Vue Project Manager`管理界面`Plugins`菜单中我添加了`cli-plugin-router`插件，在src目录下生成了`router`文件夹和`views`文件夹。启动服务时候报错：
```
$ npm run serve
> shm@0.1.0 serve D:\flask-vue-shm\shm
> vue-cli-service serve
 INFO  Starting development server...
98% after emitting CopyPlugin
 ERROR  Failed to compile with 1 error                        下午10:24:51
This dependency was not found:
* vue-router in ./src/router/index.js
To install it, you can run: npm install --save vue-router
```
根据提示运行命令即可。
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
打开浏览器可以看到Vue项目默认主页：`Welcome to Your Vue.js App`

### 添加组件
在VS code中添加文件`shm/scr/components/gump.vue`，内容如下：
```vue
<template>
  <div>
    <p>{{ msg }}</p>
  </div>
</template>
<script>
export default {
  name : 'Gump',
  data() {
    return {
      msg: 'Miracles happen every day.'
    }
  }
}
</script>
```
更新`shm/scr/router/index.js`，将/gump映射到Gump组件：
```js
import { createRouter, createWebHashHistory } from 'vue-router'
import Home from '../views/Home.vue'
import Gump from '../components/gump.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/gump',
    name: 'Gump',
    component: Gump
  },
  {
    path: '/about',
    name: 'About',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
]

const router = createRouter({
  history: createWebHashHistory(),
  routes
})

export default router
```
在`APP.vue`中加入gump内容：
```vue
<template>
  <div id="nav">
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link> |
    <router-link to="/gump">Gump</router-link>
  </div>
  <router-view/>
</template>
```
保存后刷新页面，可以看到默认的导航后有`Gump`链接，点击进入`http://localhost:8080/#/gump`页面。

## 连接前后端
### 安装axios
安装axios：
```
$ npm install axios --save
```
### 更新组件
更新gump.vue组件，示例如下：
```vue
<template>
  <div>
    <p>{{ msg }}</p>
  </div>
</template>
<script>
import axios from 'axios';

export default {
  name : 'Gump',
  data() {
    return {
      msg: 'Miracles happen every day.'
    }
  },
  methods: {
      getMessage() {
          const path = 'http://127.0.0.1:5000/gump';
          axios.get(path)
          .then((res) => {
              this.msg = res.data;
          })
          .catch((error) => {
              // eslint-disable-next-line
              console.error(error);
          })
      }
  },
  created() {
      this.getMessage();
  }
}
</script>
```
### 运行测试
启动flask：
```sh
$ python app.py
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
启动serve：
```
$ npm run serve
> shm@0.1.0 serve D:\flask-vue-shm\shm
> vue-cli-service serve
 INFO  Starting development server...
98% after emitting CopyPlugin
 DONE  Compiled successfully in 6519ms                                      下午11:52:57
  App running at:
  - Local:   http://localhost:8080/
  - Network: http://192.168.1.2:8080/
```
打开`http://127.0.0.1:5000/gump`可以看到的之前`app.py`返回的json内容：
```
Life was like a box of chocolates, you never know what you're gonna get.
```
打开`http://localhost:8080`,点击导航跳转到`http://localhost:8080/#/gump`,内容同上，也就是呈现了后端返回的数据，实现了数据交互。

## 引入Bootstrap
&#8195;&#8195;之前搞这个小项目Flask+Jinja2写了一部分，没有使用Vue.js，HTML代码也写了点，之前用的bootstrap版本是3.3.7，不知道新版有啥差异，这里还是使用3.3.7，安装示例：
```
$ npm install jquery@2.2.4
+ jquery@2.2.4
added 1 package from 1 contributor and audited 1346 packages in 18.735s
$ npm install bootstrap@3.3.7
+ bootstrap@3.3.7
added 1 package from 1 contributor and audited 1345 packages in 15.19s
```
在app的入口文件`shm/src/main.js`中导入boorstrap：
```js
import 'bootstrap/dist/css/bootstrap.css'
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```
删除`App.vue`中多余的样式，或者在`gump.vue`中添加样式测试：
```vue
<template>
  <div>
    <button type="button" class="btn btn-default" aria-label="Left Align">
      <span class="glyphicon glyphicon-home" aria-hidden="true"></span>
    </button>
    <button type="button" class="btn btn-default">Gump</button>
    <h2>{{ msg }}</h2>
  </div>
</template>
```
可以看到网页样式发生了改变。

## 补充
再次感谢CSDN博客博主的文章(Python之简)：[Flask-Vue前后端分离](https://blog.csdn.net/qq_1290259791/article/details/81174383)
