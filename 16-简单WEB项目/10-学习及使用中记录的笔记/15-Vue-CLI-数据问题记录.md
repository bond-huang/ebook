# Vue-CLI-数据问题记录
记录学习和使用过程中知识点和遇到的问题。
## 数据获取
### 数据获取问题
&#8195;&#8195;在SHM项目学习和编写过程中，发现`menu`数据格式始终获取不到，刚开始以为是Vue中获取数据的递归渲染的方式写的有问题，后来怀疑是Mock数据格式可能有问题，但是测试感觉数据没有问题。数据如下所示：
```js
    'GET /test': {
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
              path: '/modeller',
            },
            {
              menuId: "1-1-2",
              menuType: 2,
              menuName: 'AIXtest2',
              path: '/modeller',
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
                path: '/modeller',
              },
              {
                menuId: "1-2-2",
                menuType: 2,
                menuName: 'LinuxXtest2',
                path: '/modeller',
              }]
        }]
    }
```
#### 数据测试
先把数据简化：
```js
    'GET /test': {
        statusCode: "200", statusMessage: "succcess", data: 
        {
          menuId: "1-1",
          menuType: 1,
          menuName: 'AIX system',
          children: [
            {
              menuId: "1-1-1",
              menuType: 2,
              menuName: 'AIXtest1',
              path: '/modeller',
            },
            {
              menuId: "1-1-2",
              menuType: 2,
              menuName: 'AIXtest2',
              path: '/modeller',
            }]
        },
    }
```
在`Modeller.vue`中使用下面方式获取数据并查看：
```html
<ul>
    <li v-for="(value, index) in testData" :key="index">
      {{ index }}:{{ value }}
    </li>
</ul>
```
输出示例：
```
· menuId:1-1
· menuType:1
· menuName:AIX system
· children:[ { "menuId": "1-1-1", "menuType": 2, "menuName": "AIXtest1", "path": "/modeller" }, {   "menuId": "1-1-2", "menuType": 2, "menuName": "AIXtest2", "path": "/modeller" } ]
```
使用下面方式获取数据并查看：
```html
<ul>
    <li v-for="(item, index) in testData.children" :key="index">
      {{ index }}:{{ item.menuName }}
    </li>
</ul>
```
输出示例：
```
· 0:AIXtest1
· 1:AIXtest2
```
回到最开始比较完整的数据，使用下面方法获取数据：
```html
<ul>
    <div v-for="(item, index) in testData" :key="index">
        <li >{{ index }}:{{ item.menuId }}:{{ item.menuName }}</li>
    </div>
</ul>
```
输出示例：
```
· 0:1-1:AIX system
· 1:1-2:Linux system
```
添加一个条件语句：
```html
<ul>
    <div v-for="(item, index) in testData" :key="index">
        <li v-if="item.menuType == 2">
        {{ index }}:{{ item.menuId }}:{{ item.menuName }}
        </li>
    </div>
</ul>
```
输出示例：
```
· 1:1-2:Linux system
```
把数据弄到表格中示例：
```vue
<template>
  <div v-for="(item, index) in testData" :key="index">
    <el-table
      v-loading="loading"
      element-loading-text="拼命加载中"
      element-loading-spinner="el-icon-loading"
      v-if="item.menuType == 2"
      :data="item.children"
      border
      style="width: 100%; margin-top: 18px;"
    >
      <el-table-column fixed label="序号" type="index" width="60"></el-table-column>
      <el-table-column fixed prop="menuID" label="菜单ID" width="120"></el-table-column>
      <el-table-column fixed prop="menuName" label="菜单名称" width="230"></el-table-column>
      <el-table-column fixed prop="path" label="备注"></el-table-column>
      <el-table-column fixed="right" label="操作" width="160">
        <template v-slot="scope">
          <el-button @click="openColumn(scope.row)" type="text" size="small">字段管理</el-button>
          <el-button type="text" size="small" @click="toModelEdit(scope.row)">编辑</el-button>
          <el-button @click="handleDeleteModel(scope.row.modelId)" type="text" size="small">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>
```
表格中会获取到`Linux system`那组数据相关信息。

#### 问题原因
&#8195;&#8195;上面测试说明数据没有问题，数据展示的逻辑也没用问题,回到`Aside.vue`组件和`ChildrenMenu.vue`中，也就是从Mock中获取数据方式有问题，API很简单应该没问题，最后排查到了原因，是因为把`mounted()`写到了`methods`里面，错误代码示例：
```js
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
    },
    mounted(){
      getMenu().then(resp => {
        this.sideMenuList = resp
      })
    }
  }
```
把`mounted()`写错了位置，导致数据获取不到，后面处理的就都是空的数据。

## 待补充
