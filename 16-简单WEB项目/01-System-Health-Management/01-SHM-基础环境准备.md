# SHM-基础环境准备
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
$ npm install bootstrap-icons
+ bootstrap-icons@1.4.1
added 1 package from 1 contributor and audited 1347 packages in 18.91s
```
在app的入口文件`shm/src/main.js`中导入boorstrap：
```js
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-icons/font/bootstrap-icons.css'
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

## SHM-Vuex及ElementUI组件
前面内容参考CSDN博客(Python之简)，后面参考hzwy23分享的文档及各官方文档：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue CLI官方文档：[https://cli.vuejs.org/zh/](https://cli.vuejs.org/zh/)
- Vuex官方文档：[https://vuex.vuejs.org/zh/](https://vuex.vuejs.org/zh/)
- Vue Router官方文档：[https://router.vuejs.org/zh/installation.html](https://router.vuejs.org/zh/installation.html)
- Element UI官方地址:[https://element-plus.gitee.io/#/zh-CN](https://element-plus.gitee.io/#/zh-CN)
- Element UI官方中文文档:[https://element-plus.gitee.io/#/zh-CN/component/installation](https://element-plus.gitee.io/#/zh-CN/component/installation)
- 大佬hzwy23分享文档的github地址：[https://github.com/hzwy23/vue-admin](https://github.com/hzwy23/vue-admin)

### 集成Vuex组件
&#8195;&#8195;Vuex是一个专为Vue.js应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。模块工作流程图如下：    
![Vuex 模块工作流程图](https://vuex.vuejs.org/vuex.png)

#### Vuex模块核心概念
核心概念如下：
- State：存储状态变量，[https://vuex.vuejs.org/zh/guide/state.html](https://vuex.vuejs.org/zh/guide/state.html)
- Getter：获取状态变量，接受state作为其第一个参数，[https://vuex.vuejs.org/zh/guide/getters.html](https://vuex.vuejs.org/zh/guide/getters.html)
- Mutation：更改Vuex的store中的状态的唯一方法是提交mutation，[https://vuex.vuejs.org/zh/guide/mutations.html](https://vuex.vuejs.org/zh/guide/mutations.html)
- Action：提供入口方法，修改存储状态的值，[https://vuex.vuejs.org/zh/guide/actions.html](https://vuex.vuejs.org/zh/guide/actions.html)
- Module：状态模块化管理，每个模块拥有自己的state、mutation、action、getter、甚至是嵌套子模块，[https://vuex.vuejs.org/zh/guide/modules.html](https://vuex.vuejs.org/zh/guide/modules.html)

#### 集成Vuex
使用nmp安装，安装在当前项目node_modules目录下：
```
$ npm install vuex --save
+ vuex@3.6.2
added 1 package from 1 contributor and audited 1348 packages in 17.303s
```
&#8195;&#8195;或者在`Vue UI`的`Vue Project Manager`管理界面`Plugins`菜单中添加`cli-plugin-Vuex`插件，在src目录下生成了`store`文件夹，文件夹中包含文件`index.js`。

在main.js中会自动加入或者修改如下内容：
```js
import store from './store'

createApp(App).use(store).use(router).mount('#app')
```
#### Vuex使用
##### 设置state值
示例如下：
```js
//格式示例
this.$store.dispatch('action名称','state新值')
//使用示例
this.$store.dispatch('authHeight', '100px')
```
说明：
- 第一个参数是action中定义的方法，第二个参数是state新的值
- 需要调整`height`这个变量的值为`100px`。假设action中定义了一个方法`autoHeight`

##### 获取state值
通过`mapGetters`方法获取到`state`中`height`这个变量，或者从`data`中读取`clientHeight`变量:
```js
import { mapGetters } from "vuex";
export devault {
    computed: {
        //从vuex中获取浏览器高度，实时更新，保持左侧菜单栏高度与浏览器高度一致
        ...mapGetters{["height"]}
    },
    data(){
        return {
            //从vuex读取state状态的第二种方法
            clientHight: this.$store.getters.hight
        }
    }
}
```
### 集成ElementUI组件
&#8195;&#8195;Element Plus，一套为开发者、设计师和产品经理准备的基于Vue 3.0的桌面端组件库。集成了ElementUI 后，可以方便我们更快的开发出更漂亮的 Web 页面。
#### 集成ElementUI组件方法
使用npm安装：
```
$ npm install element-plus --save
+ element-plus@1.0.2-beta.41
added 8 packages from 13 contributors and audited 1344 packages in 41.515s
```
&#8195;&#8195;或者在`Vue UI`的`Vue Project Manager`管理界面`Plugins`菜单中查找并添加`vue-cli-plugin-element-plus`插件，下载后记得invoke，然后在src目录下生成了`plugins`文件夹，文件夹中包含文件`element.js`文件。

在main.js中会自动加入或者修改如下内容：
```js
import installElementPlus from './plugins/element'

const app = createApp(App)
installElementPlus(app)
app.use(store).use(router).mount('#app')
```
在App.vue中会加入如下代码：
```vue
<template>
  <img src="./assets/logo.png">
  <div>
    <p>
      If Element Plus is successfully added to this project, you'll see an
      <code v-text="'<el-button>'"></code>
      below
    </p>
    <el-button type="primary">el-button</el-button>
  </div>
  <HelloWorld msg="Welcome to Your Vue.js App"/>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>
```
&#8195;&#8195;运行后打开主页可以看到样式，示例了`Element Plus`的代码片段和按钮样式。

### 页面布局
&#8195;&#8195;之前做测试引入了bootstrap，使用也比较方便，大佬推荐了element-plus，是基于Vue 3.0的桌面端组件库，配合Vue可能更好，并且看起来比较容易上手，决定后面也没布局采用element-plus。
#### 页面整体布局
&#8195;&#8195;整体布局之前是参考bootcss中模板：[https://v3.bootcss.com/examples/dashboard/](https://v3.bootcss.com/examples/dashboard/)，使用`element-plus`后整体布局模式不变：
- Header:头部logo和导航等
- Aside:侧边菜单显示
- Main:主要内容显示区域
- Footer:底部信息

#### 整体布局示例
&#8195;&#8195;把之前测试的`Home.vue`修改成了`Vuehome.vue`，然后在`views`目录下新建`Home.vue`文件，作为项目页面主页，将路由添加到`router`文件夹下的`index.js`中。在ElementPlus官方选取了一个模板，写入`Home.vue`中：
```vue
<template>
<div class="common-layout">
  <el-container>
    <el-header>Header</el-header>
    <el-container>
      <el-aside width="250px">Aside</el-aside>
      <el-container>
        <el-main>Main</el-main>
        <el-footer>Footer</el-footer>
      </el-container>
    </el-container>
  </el-container>
</div>
</template>

<style>
  .el-header, .el-footer {
    background-color: #B3C0D1;
    color: #333;
    text-align: center;
    line-height: 60px;
  }

  .el-aside {
    background-color: #D3DCE6;
    color: #333;
    text-align: center;
    line-height: 300px;
  }

  .el-main {
    background-color: #E9EEF3;
    color: #333;
    text-align: center;
    line-height: 440px;
  }
  body > .el-container {
    margin-bottom: 40px;
  }

  .el-container:nth-child(5) .el-aside,
  .el-container:nth-child(6) .el-aside {
    line-height: 260px;
  }

  .el-container:nth-child(7) .el-aside {
    line-height: 320px;
  }
</style>
```
运行查看初步效果，页面布局以次为基准，然后依次设计各模块。
