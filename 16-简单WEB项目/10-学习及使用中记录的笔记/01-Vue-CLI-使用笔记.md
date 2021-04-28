# Vue-CLI-使用笔记
记录学习和使用过程中知识点。
## UI组件引入
&#8195;&#8195;在Vue-CLI中，通常`components`是存放组件的位置，`views`是存放页面位置。在脚手架中，`Home.vue`页面中引入了`HelloWorld.vue`组件，没用导航栏，导航栏在根组件`App.vue`中，这样所有页面都会有导航。这里想把导航作为一个组件，在需求的页面中引用。
### 组件引入示例
原本`APP.vue`中内容：
```vue
<template>
  <div id="nav">
    <router-link to="/">Home</router-link> |
    <router-link to="/login">Login</router-link> |
    <router-link to="/gump">Gump</router-link>
  </div>
  <router-view/>
</template>
```
修改成如下：
```vue
<template>
  <router-view/>
</template>
```
在`components`中新建`navbar.vue`组件，内容如下：
```vue
<template>
    <div id="nav">
    <router-link to="/">Home</router-link> |
    <router-link to="/login">Login</router-link> |
    <router-link to="/gump">Gump</router-link>
  </div>
</template>
<script>
export default {
  name: 'Navbar',
}
</script>
```
例如在`Home.vue`中进入`navbar.vue`组件，`Home.vue`内容如下：
```vue
<template>
  <div class="home">
    <Navbar/>
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import Navbar from '@/components/navbar.vue'
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'Home',
  components: {
    Navbar,
    HelloWorld
  }
}
</script>
```
说明
- 在`template`加入Navbar内容，注意格式
- 在`script`中需要import，注意名称

## 待补充
