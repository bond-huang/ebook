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
`api`目录下`login.js`代码如下：
```js
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
