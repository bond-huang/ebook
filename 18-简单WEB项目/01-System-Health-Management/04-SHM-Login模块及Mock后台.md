# SHM-Login模块及Mock后台
边学边做，学习参考文档和链接：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue CLI官方文档：[https://cli.vuejs.org/zh/](https://cli.vuejs.org/zh/)
- Bootstrap图标库：[https://icons.bootcss.com/](https://icons.bootcss.com/)
- Element-Plus官方中文文档:[https://element-plus.gitee.io/#/zh-CN/component/installation](https://element-plus.gitee.io/#/zh-CN/component/installation)
- Element-Plus component：[https://element-plus.gitee.io/#/zh-CN/component/container](https://element-plus.gitee.io/#/zh-CN/component/container)
- mockjs官方：[http://mockjs.com/](http://mockjs.com/)
- mockjs官方文档：[https://github.com/nuysoft/Mock/wiki](https://github.com/nuysoft/Mock/wiki)
- 大佬hzwy23分享文档的github地址：[https://github.com/hzwy23/vue-admin](https://github.com/hzwy23/vue-admin)

## 登录入口设计
&#8195;&#8195;登录入口主要负责接收用户账号和密码，校验用户身份。当用户身份校验通过后，进入系统。常见的几种登录方式如下所示：
- 微信扫码登录
- 支付宝扫码登录
- 手机验证码登录
- 用户账号密码登录
- 微博授权登录

&#8195;&#8195;本项目采用简单账号密码登录，后台校验用户账号和密码是否正确，如果正确，则返回有效的`token`信息， `Web`端上携带`token`信息请求后台服务。

`views`目录下`login.vue`代码如下：
```vue
<template>
  <div class="auth-body">
    <div class="container">
      <el-form class="form-auth">
        <h3>System Health Management</h3>
        <el-form-item prop="username" label="Username：">
          <el-input name="username" v-model="username" placeholder="Username" />
        </el-form-item>
        <el-form-item prop="password" label="Password: ">
          <el-input type="password" name="passwrod" v-model="password" placeholder="Password" />
        </el-form-item>
          <el-button type="primary" @click="loginSubmit">Sign in</el-button>
      </el-form>
    </div>
  </div>
</template>

<script>
import { login } from '@/api/login.js';

export default {
  name: "login",
  data() {
    return {
      username: "admin",
      password: "123456"
    };
  },
  methods: {
    loginSubmit() {
      login(this.username, this.password).then(resp => {
          if (resp) {
             this.$store.dispatch('loginStatus', true);
             this.$router.push('/dashboard')
          }
      });
    }
  }
};
</script>

<style>
.auth-body {
  background:url(../assets/background.jpg) repeat center 0px;
  background-size: cover;
  font-family: sans-serif;
  padding-top: 100px;
  padding-bottom: 150px;
}

.form-auth {
  max-width: 400px;
  padding: 15px;
  margin: 0 auto;
  background:rgba(255,255,255,0.3);		/* 背景颜色和透明度 */
  border-radius: 10px;					/* 边角弧度 */
}
.form-auth .checkbox {
  font-weight: normal;
}
</style>
```
登录的API代码如下：
```js
// @api/login.js
import axios from "axios"
import qs from 'qs'

export function login(username, password){
    return axios.post('/login', qs.stringify({
        username: username,
        password: password
    }))
}
```
## Mock数据设计
&#8195;&#8195;当开发前端Web服务时，可能后台服务尚未开发完成，此时可以借助于`Mock`服务来模拟后台服务，当`Web`端使用`Axios`向后台服务发起请求时，请求将会被`Mock`服务拦截，`Mock`服务返回预先定义的数据。当后台服务开发完成后，只需关闭对应的`Mock`服务，此时请求将会被转发到真正的后台服务。

### 安装mockjs
使用nmp安装，安装在当前项目node_modules目录下：
```
$ $ npm install mockjs --save
+ mockjs@1.1.0
updated 1 package and audited 1346 packages in 15.92s
```
### 使用mockjs
在`src`目录下新建文件夹`mock`，新建文件`mock.js`和`data`文件夹，`mock.js`文件内容代码如下：
```js
import Mock from 'mockjs'

const mock_source = ['biz.js', 'sys.js']

function load(mock_source) {
    for (let i = 0; i < mock_source.length; i++) {
        let file = import('./data/' + mock_source[i])
        file.then(content => {
            if (content && content.default) {
                initMock(content.default)
            }
        })
    }

}

function initMock(rules) {
    for (let [rule, resp] of Object.entries(rules)) {
        const element = rule.split(" ")
        if (element && element.length == 2) {
            const rtype = element[0].trim()
            const rurl = element[1].trim()
            Mock.mock(rurl, rtype.toLowerCase(), resp)
        } else {
            Mock.mock(rule, resp)
        }
    }
}

if (mock_source && mock_source.length > 0) {
    load(mock_source)
}
```
### Mock数据
&#8195;&#8195;在`data`文件夹下新建文件`sys.js`和`biz.js`文件，`sys.js`文件中主要是登录信息以及侧边导航的数据，内容如下所示：
```js
import qs from 'qs'
const menu = {
    'POST /login': function(params){
        const param = qs.parse(params.body)
        if (param.username == 'admin' && param.password == '123456') {
            return {
              statusCode: "200",
              statusMessage: 'Successful',
              data: {
                accessToken: 'xxx',
                refreshToken: 'xxx'
              }
            }
        } else {
          return {
            statusCode: "403",
            statusMessage: 'Login failed',
            data: {
              accessToken: '-',
              refreshToken: '-'
            }
          }
        }
    },
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
}

export default menu;
```
`biz.js`文件中主要是`Allsystems.vue`组件中需要获取的数据：
```js
const modeller = {
    'GET /allsystems': {
        statusCode: "200", statusMessage: "succcess", data: {
            total: 12,
            pages: 2,
            content: [{
                HostId: 1,
                HostType: "AIX",
                HostName: "AIXtest1",
                IPadd: "192.168.100.100",
                Description: "IBM AIX test system",
            },
            {
                HostId: 2,
                HostType: "AIX",
                HostName: "AIXtest2",
                IPadd: "192.168.100.101",
                Description: "IBM AIX test system",
            },
            {
                HostId: 3,
                HostType: "AIX",
                HostName: "AIXtest3",
                IPadd: "192.168.100.102",
                Description: "IBM AIX test system",
            },
            {
                HostId: 4,
                HostType: "AIX",
                HostName: "AIXtest4",
                IPadd: "192.168.100.103",
                Description: "IBM AIX test system",
            },
            {
                HostId: 5,
                HostType: "AIX",
                HostName: "AIXtest5",
                IPadd: "192.168.100.104",
                Description: "IBM AIX test system",
            },
            {
                HostId: 6,
                HostType: "AIX",
                HostName: "AIXtest6",
                IPadd: "192.168.100.105",
                Description: "IBM AIX test system",
            },
            {
                HostId: 7,
                HostType: "AIX",
                HostName: "AIXtest7",
                IPadd: "192.168.100.106",
                Description: "IBM AIX test system",
            },
            {
                HostId: 8,
                HostType: "Linux",
                HostName: "Linuxtest1",
                IPadd: "192.168.100.107",
                Description: "Red Hat Enterprise Linux",
            },
            {
                HostId: 9,
                HostType: "Linux",
                HostName: "Linuxtest2",
                IPadd: "192.168.100.108",
                Description: "Red Hat Enterprise Linux",
            },
            {
                HostId: 10,
                HostType: "Linux",
                HostName: "Linuxtest3",
                IPadd: "192.168.100.109",
                Description: "Red Hat Enterprise Linux",
            },
            {
                HostId: 11,
                HostType: "Linux",
                HostName: "Linuxtest4",
                IPadd: "192.168.100.110",
                Description: "Red Hat Enterprise Linux",
            },
            {
                HostId: 12,
                HostType: "Linux",
                HostName: "Linuxtest5",
                IPadd: "192.168.100.111",
                Description: "Red Hat Enterprise Linux",
            },]
        }
    },
}

export default modeller;
```
## 结束
&#8195;&#8195;到现在为止，基本的框架搭建完成，后续主要是功能设计，根据自己的项目需求来添加相应的功能，前端设计完成后，还有后端开发，学习的过程还很漫长。
