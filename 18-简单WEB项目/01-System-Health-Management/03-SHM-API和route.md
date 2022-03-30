# SHM-API和route
边学边做，学习参考文档和链接：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue CLI官方文档：[https://cli.vuejs.org/zh/](https://cli.vuejs.org/zh/)
- Bootstrap图标库：[https://icons.bootcss.com/](https://icons.bootcss.com/)
- Element-Plus官方中文文档:[https://element-plus.gitee.io/#/zh-CN/component/installation](https://element-plus.gitee.io/#/zh-CN/component/installation)
- Element-Plus component：[https://element-plus.gitee.io/#/zh-CN/component/container](https://element-plus.gitee.io/#/zh-CN/component/container)
- 大佬hzwy23分享文档的github地址：[https://github.com/hzwy23/vue-admin](https://github.com/hzwy23/vue-admin)

## 集成Axios
&#8195;&#8195;Axios是一个基于promise的HTTP库,可以用在浏览器和node.js中，之前基础环境准备中已经安装和演示了。安装不作介绍，再次学习演示下。

### 安装Axios
&#8195;&#8195;（后面卸载了此插件）在`Vue UI`的`Vue Project Manager`管理界面`Plugins`菜单中添加`vue-cli-plugin-axios`插件，在src目录下`plugins`文件夹中生成文件`axios.js`。文件内容如下：
```js
"use strict";

import Vue from 'vue';
import axios from "axios";

// Full config:  https://github.com/axios/axios#request-config
// axios.defaults.baseURL = process.env.baseURL || process.env.apiUrl || '';
// axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
// axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';

let config = {
  // baseURL: process.env.baseURL || process.env.apiUrl || ""
  // timeout: 60 * 1000, // Timeout
  // withCredentials: true, // Check cross-site Access-Control
};

const _axios = axios.create(config);

_axios.interceptors.request.use(
  function(config) {
    // Do something before request is sent
    return config;
  },
  function(error) {
    // Do something with request error
    return Promise.reject(error);
  }
);

// Add a response interceptor
_axios.interceptors.response.use(
  function(response) {
    // Do something with response data
    return response;
  },
  function(error) {
    // Do something with response error
    return Promise.reject(error);
  }
);

Plugin.install = function(Vue, options) {
  Vue.axios = _axios;
  window.axios = _axios;
  Object.defineProperties(Vue.prototype, {
    axios: {
      get() {
        return _axios;
      }
    },
    $axios: {
      get() {
        return _axios;
      }
    },
  });
};

Vue.use(Plugin)

export default Plugin;
```
同时，修改了文件`main.js`里面内容：
```js
import './plugins/axios'
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-icons/font/bootstrap-icons.css'
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
import installElementPlus from './plugins/element'

const app = createApp(App)
installElementPlus(app)
app.use(store).use(router).mount('#app')
```
运行项目后报了一个错：
```
error  'options' is defined but never used  no-unused-vars
```
暂时删掉`options`，又报了一个：
```
"export 'default' (imported as 'Vue') was not found in 'vue'
```
版本原因，使用格式应该是：
```js
import { createApp } from 'vue'
```
&#8195;&#8195;版本不匹配，最后放弃了此插件，卸载了`vue-cli-plugin-axios`插件，使用之前安装的，根据大佬的文档指导内容写入Axios代码初始化，在`plugins`文件夹中`axios.js`文件写入代码示例：
```js
import axios from 'axios';
import { ElMessage } from 'element-plus';
// axios.defaults.baseURL = process.env.VUE_APP_BASE_API;
// axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';

axios.interceptors.request.use(config => {
    return config;
}, error => {
    ElMessage.error(error)
})

axios.interceptors.response.use(response => {
    if (response && response.status == 200) {
        if (response.data.statusCode == "200") {
            return response.data.data;
        } else {
            ElMessage.error(response.data.statusMessage)
            return false;
        }
    }
    ElMessage.error('Request API failed');
}, error => {
    ElMessage.error(error)
    return false;
})
```

## 权限拦截管理
&#8195;&#8195;权限管理就是不同用户拥有不同执行权限，在系统管理中很重要，不作详细介绍，权限管理通常分为几个形式：
- API权限控制
- 数据权限控制
- 前端页面权限控制
    - 在每个页面中加入判断，校验用户是否登录
    - 全局校验用户是否登录
- 页面按钮权限控制

&#8195;&#8195;可以使用Vue Router导航守卫功能来设置全局的用户登录校验。导航守卫主要在如下几个阶段来拦截请求。分别是：
- beforeEach 全局前置守卫，在导航路由触发前拦截
- beforeRouteUpdate
- beforeEnter
- beforeRouteEnter
- beforeResolve
- afterEach

使用beforeEach实现登录校验的判断，在`src`目录下新建文件`permission.js`，代码示例如下：
```js
import router from '@/router/index.js';
import store from '@/store/index.js';

function checkLogin() {
    // todo 校验用户是否登录，以及用户 token 令牌是否过期
    return !store.getters.loginStatus;
}
// @param to 到哪里去，跳转到哪个路由
// @param from 从哪里来，从哪个路由跳转过来的
// @param next 执行跳转
router.beforeEach((to, from, next) => {
    // 判断用户是否登录，
    // 如果用户已经登录，执行 next() 方法，
    // 如果用户未登录，则跳转到登录页面
    if (to.path != '/login' && checkLogin()) {
        next({ path: '/login' })
    } else {
        next()
    }
})
```
说明：
- 每次触发路由导航时，都会执行这个判断逻辑，当发现用户未登录，或登录的`token`失效后，将会被引导进入登录页面。注意一定要判断目标地址是否为登录地址，否则出现死循环。
- `checkLogin`方法现在只是读取了保存在`vuex`中的用户登录状态，这种处理方式并不适合生产环境。：在浏览器执行`F5`刷新时，`Vuex`存储的状态值会丢失，用户要重新登录
- 通常生产环境中，会将`token`信息存储到`Cookies`中，登录状态存储在`Vuex`中，判断用户是否登录要结合`Cookies`中的`token`与`Vuex`中的登录状态一起判断。如用户登录状态为`false`时，从`Cookies`中读取用户`token`值，向后台服务请求验证`token`有效性，如果`token`有效，则设置`Vuex`中用户登录状态为`true`，然后跳转到目标路由地址。否则跳转到登录页面
- 再次运行项目，发现路由已经跳转到之前做测试的登录页面中去了

## 菜单后台加载设计
字段介绍
- menuId菜单编码，每个菜单必须对应一个唯一的菜单编码。
- menuName 菜单名称。
- menuType 菜单类型。1 表示目录类型，2 表示叶子菜单，目录类型菜单，表示其下还有菜单信息，叶子类型菜单，表示达到树底部。
- path 前端路由值，叶子菜单被点击时，将会跳转到该路由。目录菜单设置path值将会被忽略。
- iconCLass 目录类型菜单前边的小图标。
- children 表示目录类型菜单下的子菜单信息。

示例创建一个目录菜单，目录菜单下边挂载两个叶子菜单：
```js
'GET /menu': {
      statusCode: "200", statusMessage: "succcess", data: 
      [{
        menuId: "1-1",
        menuType: 1,
        menuName: 'AIX system',
        children: [
          {
            menuId: "1-1-1",
            menuType: 2,
            menuName: 'AIXtest1',
            path: '/allsystems',
          },
          {
            menuId: "1-1-2",
            menuType: 2,
            menuName: 'AIXtest2',
            path: '/allsystems',
          }]
      },
      {
          menuId: "1-2",
          menuType: 2,
          menuName: 'Linux system',
          children: [
            {
              menuId: "1-2-1",
              menuType: 2,
              menuName: 'Linuxtest1',
              path: '/allsystems',
            },
            {
              menuId: "1-2-2",
              menuType: 2,
              menuName: 'LinuxXtest2',
              path: '/allsystems',
            }]
      }]
  }
```
在Web中向后台请求菜单信息的方式：
```js
// api/menu.js
import axios from "axios"

export function getMenu(){
    return axios.get('/menu')
}
```
## 前端路由设计
&#8195;&#8195;在`router`目录下`index.js`文件中，有之前做测试的路由，全段路由一般由下面几个部分组成：
- path：路由地址
- name：路由名称
- component：路由对应的组件
- meta：路由元数据，如路由标签等
- children：子路由信息

在程序中便可以调用路由跳转方法进行路由切换操作：
```js
// 字符串
router.push('/foo')
// 对象
router.push({ path: '/foo' })
// 命名的路由
router.push({ name: 'foo', params: { userId: '123' }})
// 带查询参数，变成 /register?plan=private
router.push({ path: '/foo', query: { plan: 'private' }})
```
在HTML页面元素中实现路由跳转的方法：
```html
<router-link to="/foo">Go to Foo</router-link>
```
在浏览历史记录中切换跳转方法：
```js
router.go(n)
// n 表示第几个历史记录，n 必须是整数
```
### 基础页面
&#8195;&#8195;我在组件目录下新建了`layout`目录，里面包含`BaseLayout.vue`和`EmptyLayout.vue`两个组件，`BaseLayout.vue`内容如下：
```vue
<template>
  <div class="common-layout">
    <el-container direction="vertical">
      <Header/>
      <el-container>
        <Aside/>
        <el-container style="border: 1px solid #eee">
          <el-main><Content/></el-main>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>

<script>
import Header from '@/components/Header/Header.vue'
import Aside from '@/components/Aside/Aside.vue'
import Content from '@/components/Main/Content.vue'

export default {
  name: 'BaseLayout',
  components: {
    Header,
    Aside,
    Content
  }
}
</script>

<style>
  .el-main {
    background-color: #E9EEF3;
    color: #333;
    text-align: center;
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
即之前测试中`Home.vue`页面中内容，`EmptyLayout.vue`内容如下：
```vue
<template>
  <router-view></router-view>
</template>
```
### 前端路由
前端路由往往嵌套多层，`router`目录下`index.js`文件写入路由信息，路由信息如下所示：
```js
import { createRouter, createWebHashHistory } from 'vue-router'
import BaseLayout from '@/components/layout/BaseLayout'
import EmptyLayout from '@/components/layout/EmptyRouter'
import Dashboard from '@/views/Dashboard'
import Login from '@/views/Login'
import Vuehome from '@/views/Vuehome.vue'
import Gump from '@/views/Gump.vue'

import Allsystems from '@/views/allsystems/AllSystems'
import HostUpdate from '@/views/allsystems/HostUpdate'

const routes = [{
  path: '',
  component: EmptyLayout,
  redirect: 'dashboard',
  children: [{
      path: '/login',
      component: Login,
      name: 'login',
      meta: {
          title: 'login'
      }
  }]
}, {
  path: '',
  component: BaseLayout,
  redirect: 'dashboard',
  children: [{
      path: 'dashboard',
      component: Dashboard,
      name: 'dashboard',
      meta: {
          title: 'home'
      }
  }]
},
  {
    path: '/gump',
    component: BaseLayout,
    children: [{
        path: '/gump',
        component: Gump,
        name: 'gump',
        meta: {
            title: 'gump'
        }
    }]
  },
  {
    path: '/vuehome',
    component: BaseLayout,
    children: [{
        path: '/vuehome',
        component: Vuehome,
        name: 'vuehome',
        meta: {
            title: 'vuehome'
        }
    }]
  },
  {
    path: '/allsystems',
    component: BaseLayout,
    children: [{
        path: '/allsystems',
        component: Allsystems,
        name: 'allsystems',
        meta: {
            title: 'All Systems'
        }
    },
    {
        path: '/allsystems',
        component: EmptyLayout,
        meta: {
            title: 'All Systems'
        }, children: [
            {
                path: 'update',
                name: 'update',
                component: HostUpdate,
                meta: {
                    title: 'Update Host',
                }
            },
        ]
    }]
  },
]

const router = createRouter({
  history: createWebHashHistory(),
  routes
})

export default router
```
说明：
- 首先打开`BaseLayout`组件，然后在这个组件中找到&#60;router-view>&#60;/router-view>
- 然后打开`EmptyLayout`组件，`EmptyLayout`组件被嵌入`BaseLayout`组件的&#60;router-view>&#60;/router-view>内
- 最后打开`Allsystems`组件，此时其上级组件`EmptyLayout `组件内查找&#60;router-view>&#60;/router-view>，然后将`Allsystems`组件嵌入到`EmptyLayout`组件的&#60;router-view>&#60;/router-view>内
- 前端路由定义时，如果路由中包含了`children`属性，那么这个组件内一定带有&#60;router-view>&#60;/router-view> DOM元素 ，否则`children`内的组件无处安放

整个项目的入口组件`App.vue`,里边就是一个&#60;router-view>&#60;/router-view>，代码如下：
```vue
<template>
  <router-view/>
</template>
```
## 结束
