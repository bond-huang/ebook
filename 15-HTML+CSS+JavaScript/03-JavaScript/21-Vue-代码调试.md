# Vue-代码调试
参考官方文档学习及使用过程中记录的笔记。
## VScode中调试
参考文档：
- [Vue.js官方文档-在VS Code中调试](https://cn.vuejs.org/v2/cookbook/debugging-in-vscode.html)

### 先决条件
在VScode中安装相应插件,我使用的Firefox，链接：
- [Debugger for Firefox](https://marketplace.visualstudio.com/items?itemName=firefox-devtools.vscode-firefox-debug)

### 在浏览器中展示源代码
&#8195;&#8195;在`VS Code`调试`Vue`组件之前，需要更新`webpack`配置以构建`source map`。之后，调试器就有机会将一个被压缩的文件中的代码对应回其源文件相应的位置。项目根目录下创建`vue.config.js`文件，写入代码示例如下：
```js
module.exports = {
    configureWebpack: {
      devtool: 'source-map'
    }
  }
```
如果写错到了bable.config.js文件里面，运行项目会报错：
```
 ERROR  Error: Unknown option: .configureWebpack. Check out https://babeljs.io/docs/en/babel-core/#options for more information about options.
```
### 从VS Code启动应用
&#8195;&#8195;点击左侧`Activity Bar`里的`Run and Debug`图标来到`Debug`视图(Ctrl Shift D)，然后点击`Create a launch.json file`来配置一个`launch.json`的文件，选择`Firefox`环境。然后将生成的`launch.json`的内容替换成为相应的配置：
```json
{
    "version": "0.2.0",
    "configurations": [
      {
        "type": "firefox",
        "request": "launch",
        "name": "vuejs: firefox",
        "url": "http://localhost:8080",
        "webRoot": "${workspaceFolder}/src",
        "pathMappings": [{ "url": "webpack:///src/", "path": "${webRoot}/" }]
      }
    ]
  }
```
### 使用示例
以System-Health-Management项目为例，出现问题时获取不到菜单数据。
#### 设置断点
在`ChildrenMenu.vue`组件中，第二行代码：
```
<div v-for="(item, index) in children" :key="index">
```
在其前面设置代码，看看`children`内容，设置后运行项目：
```
$ npm run serve
```
&#8195;&#8195;回到`Debug`视图，选择`vuejs:firefox`配置，然后按`F5`或点击绿色的`play按钮`。会自动弹出新的浏览器实例打开`http://localhost:8080`，设置的断点现在被命中了。

#### 故障排查
&#8195;&#8195;在`Debug`视图的右下方的`DEBUG CONSOLE`中，会有相关的信息，看是否满足代码的需求的数据。如果此处判断不了故障原因，设置其它断点，继续排查。

此次断点看到的内容：
```
Array(1) ["1"]
0:"1"
length:1
__proto__:Array(0)
```
再次设置断点，运行后感觉卡住不动`DEBUG CONSOLE`中会有如下信息：
```
[HMR] Waiting for update signal from WDS...
```
找到`node_modules->webpack->hot`文件夹下`log.js`文件，使用`Ctrl+/`注释部分内容，示例：
```js
module.exports = function(level, msg) {
	// if (shouldLog(level)) {
		// if (level === "info") {
			// console.log(msg);
		// } else if (level === "warning") {
			// console.warn(msg);
		// } else if (level === "error") {
			// console.error(msg);
		// }
	// }
};
```
参考链接：[[HMR] Waiting for update signal from WDS…](https://stackoverflow.com/questions/59695102/reactjs-console-error-hmr-waiting-for-update-signal-from-wds)

&#8195;&#8195;设置后这条等待信息没有了，运行后有时可以看到输出有时等半天也没结果。后来又查了查，这个不是错误，只是说当您保存文件时它已准备好刷新，因此无需手动刷新。 当然我试了试手动刷新，好像也行。

参考链接：
- [Waiting for update signal from WDS](https://stackoverflow.com/questions/60192271/waiting-for-update-signal-from-wds)
- [Vue.js官方文档:runtimeCompiler](https://cli.vuejs.org/zh/config/#runtimecompiler)

获取菜单问题最后排查到了原因，是因为把`mounted()`写到了`methods`里面，错误代码示例：
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

## Vue Devtools

## 待补充
