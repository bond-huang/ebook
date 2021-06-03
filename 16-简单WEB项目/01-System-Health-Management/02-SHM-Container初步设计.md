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

### Menu.vue设计
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
    <el-dropdown @command="handleCommand">
      <span class="el-dropdown-link">
        <i class="bi bi-person-circle"></i>
        {{userInfo.nickname}}
        <i class="el-icon-arrow-down el-icon--right"></i>
      </span>
      <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item command="1">User Admin</el-dropdown-item>
        <el-dropdown-item command="2" divided>Logout</el-dropdown-item>
      </el-dropdown-menu>
      </template>
    </el-dropdown>
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
              Gump</router-link></el-dropdown-item>
          <el-dropdown-item>
            <a href="https://github.com/bond-huang" target="_blank"> 
            <i class="bi bi-github"></i>&nbsp;GitHub</a></el-dropdown-item>
          <el-dropdown-item>
            <a href="https://github.com/bond-huang/shm/blob/master/LICENSE" target="_blank">
            <i class="el-icon-s-check"></i>&nbsp;License</a></el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
    </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  name: 'Tools',
  computed: {
    ...mapGetters(["userInfo"])
  },
  methods: {
    handleCommand(command) {
      console.log(command)
      switch(command) {
        case "1":
          break;
        case "2":
          this.logout();
      }
    },
    logout() {
        this.$confirm('Are you sure logout?', 'Prompt information', {
          confirmButtonText: 'Confirm',
          cancelButtonText: 'Cancel',
          type: 'warning'
        }).then(() => {
          this.$store.dispatch('loginStatus', false)
          this.$router.push('/login')
        });
    }
  }
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
- 代码中使用`mapGetters`获取了登录用户信息，后续再添加用户管理界面

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
&#8195;&#8195;替换后刷新页面发现布局混乱，原因在官方文档`Container Attributes`中有说明：子元素中有 `el-header`或`el-footer`时为`vertical`，否则为`horizontal`，加上属性`direction="vertical"`即可。
## Aside菜单栏设计
&#8195;&#8195;在`component`目录下新建了`Aside`目录，用于存放侧边导航的相关组件。由两个组件组成，分别是：
- Side.vue：左侧菜单栏主要布局，建构一个紧靠浏览器左侧，可以通过按钮展开或者收起，垂直方向自适应浏览器高度的区域
- ChildrenMenu.vue：显示具体的菜单信息，嵌套在Side.vue组件中

### Side.vue设计
组件`Side.vue`代码示例如下：
```vue
<template>
  <div class="shm-aside">
    <el-radio-group v-model="isCollapse">
      <el-button type="text" @click="click()">
      <i v-bind:class="foldicon"></i></el-button>
    </el-radio-group>
  <el-menu 
    default-active="1-1-1"
    class="el-menu-vertical-demo"
    @open="handleOpen" @close="handleClose" 
    :collapse="isCollapse" 
    background-color="#545c64"
    text-color="#fff"
    active-text-color="#ffd04b">
    <el-submenu index="1">
      <template #title>
        <i class="el-icon-menu"></i>
        <span>Systems Class</span>
      </template>
        <ChildrenMenu v-bind:menuData="sideMenuList" />
    </el-submenu>
    <el-submenu index="2">
      <template #title><i class="el-icon-bank-card"></i><span>System Admin</span></template>
        <el-menu-item-group>
          <el-menu-item index="2-1" @click="openPage('/allsystems')">
            All Systems
          </el-menu-item>
          <el-menu-item index="2-2" @click="openPage('/allsystems')">
            System Class
          </el-menu-item>
      </el-menu-item-group>
    </el-submenu>
    <el-submenu index="3">
      <template #title><i class="el-icon-setting"></i><span>Setting</span></template>
      <el-menu-item-group>
        <el-menu-item index="3-1">User Setting</el-menu-item>
        <el-menu-item index="3-2">Other Setting</el-menu-item>
      </el-menu-item-group>
    </el-submenu>
    <el-menu-item index="4">
      <i class="el-icon-document"></i>
      <template #title>Document</template>
    </el-menu-item>
    <el-menu-item index="5">
      <i class="el-icon-question"></i>
      <template #title>Help</template>
    </el-menu-item>
  </el-menu>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import { getMenu } from '@/api/menu.js';
import ChildrenMenu from '@/components/Aside/ChildrenMenu.vue';

export default {
  name: 'Aside',
  computed: {
    ...mapGetters(["height"])
  },
  components: {
    ChildrenMenu
  },
  data() {
    return {
    sideMenuList: [],
    isCollapse: true,
    foldicon:'el-icon-s-unfold'
    };
  },
  methods: {
    openPage(url) {
      this.$router.push(url);
    }, 
    handleOpen(key, keyPath) {
      console.log(key, keyPath);
      },
    handleClose(key, keyPath) {
      console.log(key, keyPath);
      },
    click:function(){
      if(this.isCollapse){
        this.foldicon = 'el-icon-s-fold';
      }else{
        this.foldicon = 'el-icon-s-unfold';
        }
      this.isCollapse = !this.isCollapse;
    }
  },
  mounted(){
    getMenu().then(resp => {
      this.sideMenuList = resp
    })
  }
}
</script>

<style scoped>
.shm-aside {
  float: left;
  text-align: left;
  padding: 2px 0px;
  line-height: 30px;
  background-color: #545c64;
}
.el-menu-vertical-demo:not(.el-menu--collapse) {
  width: 200px;
  overflow-y: auto;
  overflow-x: hidden;
  }
</style>
```
说明：
- 示例中从vuex中获取浏览器高度，实时更新，保持左侧菜单栏高度与浏览器高度一致
- 示例中`mounted()`作用是定时获取后台菜单信息
- 菜单展开收起的按钮比较小，颜色也不顺眼，回头有空再调整

### ChildrenMenu.vue设计
组件`ChildrenMenu.vue`代码示例如下：
```vue
<template>
  <div v-for="(item, index) in menuData" :key="index">
    <el-submenu :index="item.menuId">
      <template #title>{{ item.menuName }}</template>
        <div v-for="(item, index) in item.children" :key="index">
          <el-menu-item v-if="item.menuType == 2" :index="item.menuId" @click="openPage(item.path)">
            {{ item.menuName }}
          </el-menu-item>
        </div>
    </el-submenu>
  </div>
</template>

<script>
export default {
  props: ["menuData"],
  name: 'ChildrenMenu',
  methods: {
    openPage(url) {
      this.$router.push(url);
    }
  }
}
</script>
```
说明：
- `ChildrenMenu`组件用来渲染树形层级菜单
- 刚开始树形菜单采用递归渲染的方式实现，即在`ChildrenMenu`组件内嵌套使用组件自身，从而实现递归渲染树形菜单的效果，后来改用简单便于理解的两个for循环
- 如果`Vue.js`中组件采用递归时，该组件一定要设置`name`属性，否则递归无效
- 通过`Axios`组件向后台服务发起请求，获取左侧菜单栏信息

### 主页导入Aside
参照之前方法导入即可，后面再统一示例代码。
## Main Content部分设计
### Content.vue设计
组件`Content.vue`代码示例如下：
```vue
<template>
  <div>
    <el-card>
      <Breadcrumb></Breadcrumb>
    </el-card>
    <div class="shm-content" :style="{height: (height-120)+'px'}">
      <section style="overflow: auto !important">
        <transition name="fade" mode="out-in">
          <keep-alive>
            <el-card
              style="overflow: auto !important"
              :style="{height: (height-140)+'px'}">
              <router-view></router-view>
            </el-card>
          </keep-alive>
        </transition>
      </section>
    </div>
  </div>
</template>
<script>
import { mapGetters } from "vuex";
import Breadcrumb from "@/components/Main/Breadcrumb.vue";

export default {
  name: "Content",
  components: {
    Breadcrumb
  },
  computed: {
    ...mapGetters(["height"])
  },
}
</script>

<style scoped>
.shm-content {
  border: #f6f3f3 solid 1px;
  background-color: #f6f3f3;
  overflow-y: auto;
  padding: 6px 6px;
}
</style>
```
说明：
- 在MaContent中加入了```<router-view></router-view>```DOM元素。当左侧菜单栏点击打开相应页面后，其对应的组件将会在`Main Content`中```<router-view></router-view>```DOM元素内显示
- 示例中我手动加了一个表格进行演示，表格数据重复了10次，后期放在其它页面
- 示例中导入了`Breadcrumb`组件，是面包屑导航功能，后面介绍
- 通过给`table`传入`span-method`方法实现合并行或列，示例中使用`arraySpanMethod`方法合并了行

### Breadcrumb.vue设计
组件`Breadcrumb.vue`代码示例如下：
```vue
<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">Home</el-breadcrumb-item>
      <el-breadcrumb-item
        v-for="(item,index) in breadcrumb"
        :to="{path: item.path}"
        :key="index"
      >{{item.title}}</el-breadcrumb-item>
    </el-breadcrumb>
  </div>
</template>

<script>
export default {
  data() {
    return {
      breadcrumb: []
    };
  },
  watch: {
    $route(to) {
      const routers = to.matched;
      this.breadcrumb = [];
      if (routers && routers.length > 0) {
        for (let i = 1; i < routers.length; i++) {
          this.breadcrumb.push({
            title: routers[i].meta.title,
            path: routers[i].path
          });
        }
      }
    }
  }
};
</script>
```
说明：
- 内容基本copy大佬hzwy23内容，个人喜欢使用图标分隔符，后续再根据需要更改内容
- 前端路由往往由多层路由组成，在页面跳转过程中，引入面包屑导航功能后，可以很方便的知道当下所在的页面
- `watch`用来监听路由跳转，每次发生路由跳转时，`$route`中的值都会发生变化，`to.matched`用来获取匹配成功的路由信息

## Footer部分设计
在`component`目录下新建了`Footer`目录，用于存放Footer相关组件，`Footer.vue`内容如下：
```vue
<template>
    <div>Copyright © 2021 @vue/cli+ElementUI</div>
</template>
<script>
export default {
  name: "Footer"
};
</script>
```
说明：
- Footer主要显示一些版权信息等
- 样式放在主页实现了，主页中`el-footer`样式即是
- 在此次项目中，演示完Footer后删除了，没什么用，占位置

## 结束
后期根据自己的需求调整样式及数据类型等。
