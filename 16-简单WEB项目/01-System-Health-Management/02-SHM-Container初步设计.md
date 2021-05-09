# SHM-Container初步设计
边学边做，学习参考文档和链接：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue CLI官方文档：[https://cli.vuejs.org/zh/](https://cli.vuejs.org/zh/)
- Bootstrap图标库：[https://icons.bootcss.com/](https://icons.bootcss.com/)
- Element-Plus官方中文文档:[https://element-plus.gitee.io/#/zh-CN/component/installation](https://element-plus.gitee.io/#/zh-CN/component/installation)
- Element-Plus component：[https://element-plus.gitee.io/#/zh-CN/component/container](https://element-plus.gitee.io/#/zh-CN/component/container)
- 大佬hzwy23分享文档的github地址：[https://github.com/hzwy23/vue-admin](https://github.com/hzwy23/vue-admin)

## Header设计
在`component`目录下新建了`Header`目录，用于存放header相关的组件，目录内容如下：
- Header.vue：Header的主组件，集成其它子组件
- Logo.vue：header左侧项目logo或者主标题等
- Menu.vue：header中间的主菜单
- Tools.vue：header右侧的登录信息或其它信息

### Header.vue设计
组件`Header.vue`代码示例如下：
```vue
<template>
  <div class="shm-header">
    <el-row>
      <el-col :span="6">
        <Logo/>
      </el-col>
      <el-col :span="12">
        <Menu/>
      </el-col>
      <el-col :span="6">
        <Tools/>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Logo from '@/components/Header/Logo.vue'
import Menu from '@/components/Header/Menu.vue'
import Tools from '@/components/Header/Tools.vue'

export default {
  name: 'Header',
  components: {
    Logo,
    Menu,
    Tools
  }
}
</script>

<style scoped>
.shm-header {
  height: 50px;
  width: 100%;
  padding: 2px 0px;
  line-height: 50px;
  background-color:#337ab7;
}
</style>
```
说明：
- 使用`el-col`将header按6、12、6分成了三份
- 示例中import了三个子组件，同时将`Header`export
- 高度50px个人觉得差不多了，同时`line-height`也要设置对应的值，否则文字内容不居中
- 背景采用经典深蓝色`#337ab7;`，文字颜色在各子模块中定义

### Logo.vueS设计
组件`Logo.vue`代码示例如下：
```vue
<template>
  <div class="shm-logo"><strong>System Health Management</strong></div>
</template>

<script>
export default {
  name: 'Logo'
};
</script>

<style scoped>
.shm-logo {
  float: left;
  padding-left: 20px;
  font-family: "Microsoft YaHei";
  font-size: 2rem; 
  color: #FFFFFF;
}
</style>
```
说明：
- 暂时没想到用啥logo，只写了项目名称，并加粗显示
- 靠左对齐，左边空20px，字体为微软雅黑，大小为2rem，颜色为纯白色

### Menu.vueS设计
组件`Menu.vue`代码示例如下：
```vue
<template>
    <div class="shm-menu">
    <i class="bi bi-house-fill"></i>
        Home
    </div>
</template>

<script>
export default {
  name: 'Menu'
};
</script>

<style>
.shm-menu {
  float: left;
  padding-left: 20px;
  color: #FFFFFF;
  font-size: 15px;
}
</style>
```
说明：
- 目前只有一个Home的选项，后期在添加

### Tools.vueS设计
组件`Tools.vue`代码示例如下：
```vue
<template>
    <div class="shm-tools">
    <span class="el-dropdown-link">
    <i class="bi bi-person-circle"></i>
        Login
    </span>
    &nbsp;&nbsp;
    <el-dropdown>
      <span class="el-dropdown-link">
        More<i class="el-icon-arrow-down el-icon--right"></i>
      </span>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item>
            <router-link to="/vuehome">
              <span class="glyphicon glyphicon-chevron-down"></span>
              &nbsp;Vuehome</router-link></el-dropdown-item>
          <el-dropdown-item>
            <router-link to="/gump"><i class="bi bi-google"></i>
              &nbsp;Gump</router-link></el-dropdown-item>
          <el-dropdown-item>
            <a href="https://github.com/bond-huang" target="_blank">
            <i class="bi bi-github"></i>&nbsp;GitHub</a></el-dropdown-item>
          <el-dropdown-item>
            <a href="https://github.com/bond-huang/System-Health-Check/blob/main/LICENSE" target="_blank">
            <i class="el-icon-s-check"></i>&nbsp;License</a></el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
    </div>
</template>

<script>
export default {
  name: 'Tools'
};
</script>

<style>
.shm-tools {
  float: right;
  padding-right: 20px;
  color: #FFFFFF;
  font-size: 15px;
}
.el-dropdown-link {
  cursor: pointer;
  color: #FFFFFF;
}
</style>
```
说明：
- 在使用超链接时候，刚开始打算用Element-plus的`el-link`，但是我想点击后打开新的页面，`target`属性使用不行，可能有其它的但是我不知道，官方文档找了下没找到，最后还是使用`a`标签
- 使用`el-link`无下划线的格式示例：`el-link :underline="false" href="https://github.com/bond-huang"`

### 主页导入Header
原本的header样式`el-header`去掉，加入如下内容：
```vue
<script>
import Header from '@/components/Header/Header.vue'

export default {
  name: 'Home',
  components: {
    Header
  }
}
</script>
```
将下面代码删除，替换成对应`Header`组件：
```html
<el-header>Header</el-header>
```
## Aside菜单栏设计
