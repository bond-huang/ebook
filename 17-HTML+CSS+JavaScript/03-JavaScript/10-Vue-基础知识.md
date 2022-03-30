# Vue-基础知识
学习过程中记录的笔记，学习教程及参考链接：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue.js runoob网站教程:[https://www.runoob.com/vue2/vue-tutorial.html](https://www.runoob.com/vue2/vue-tutorial.html)

使用在线工具：
- [runoob.com菜鸟教程在线编辑器](https://www.runoob.com/try/try.php?filename=vue2-hw)

## 安装及使用方法
### 安装
安装说明：[https://cn.vuejs.org/v2/guide/installation.html](https://cn.vuejs.org/v2/guide/installation.html)
### 使用方法
可以下载安装后使用，也可以直接在线调用：
```html
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
```
### 浏览器JavaScript控制台
打开及使用方法：
- FireFox和Chrome浏览器可以按`F12`打开JavaScript控制台，或者按`Ctrl+Shift+i`组合键。
- 打开后点击`Console`选项，即可进入JavaScript控制台
- 在JavaScript控制台可以输入相应的命令，例如输入`app1.message='New message'`可以改变app1中message的内容
- 在`Accessibility`选项中可以查看网页整体架构

## 声明式渲染
### 绑定文本
Vue.js的核心是允许采用简洁的模板语法来声明式地将数据渲染进DOM的系统：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue声明式渲染示例</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app">
        <p>{{ message }}</p>
    </div>
<script>
new Vue({
    el:'#app',
    data: {
        message: 'Miracles happen every day!'
    }
})
</script>
</body>
</html>
```
示例说明：
- 示例中，script中的`Vue`的`V`需要大写，否则无效
- 修改`app.message`的值，HTML中`{{ message }}`相应的更新

### 绑定元素
示例如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue声明式渲染示例2</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app-1">
        <span v-bind:title="message">
        鼠标悬停几秒钟查看动态绑定的提示信息
        </span>
    </div>
<script>
var app1 = new Vue({
    el:'#app-1',
    data: {
        message: '页面加载于 ' + new Date().toLocaleString()
    }
})
</script>
</body>
</html>
```
示例说明：
- 示例中鼠标指向文本，悬停几秒就可以看到提示信息
- 提示信息中`new Date().toLocaleString()`是获取的页面加载日期和时间
- 注意，id选择器选择时候选择script中el里面的内容
- 打开浏览器的JavaScript控制台，输入`app1.message='New message'`，可以看到绑定了`title attribute`的HTML已经进行了更新

## 条件与循环
### 条件
例如控制一个元素是否可见：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue条件示例</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app-2">
        <p v-if="seen">不可见的</p>
    </div>
<script>
var app2 = new Vue({
    el:'#app-2',
    data: {
        seen: false
    }
})
</script>
</body>
</html>
```
示例说明：
- 示例中的元素是不可见的，data中修改成`seen:true`，就可见了
- 在浏览器的JavaScript控制台，输入`app2.seen= true`也可以让元素可见

### 循环
指令`v-for`可以绑定数组的数据来渲染一个项目列表：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue循环示例</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app-3">
        <ol>
            <li v-for="item in items">
            {{ item.name }}
            </li>
        </ol>
    </div>
<script>
var app3 = new Vue({
    el:'#app-3',
    data: {
        items:[
            { name:'Captain America'},
            { name:'Wonder Woman'},
            { name:'Iron Man'}
        ]
    }
})
</script>
</body>
</html>
```
示例说明：
- 在items列表中，每一项间隔注意加上逗号
- 在浏览器的JavaScript控制台，输入`app3.items.push({ name: 'Thor' })`，浏览器输出列表最后添加了一个新项目

## 处理用户输入
### 指令v-on
为了让用户和应用进行交互，可以用`v-on`指令添加一个事件监视器，通过它调用在Vue实例中定义的方法：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue反转消息示例</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app-4">
        <p>{{ message }}</p>
        <button v-on:click="reverseMessage">反转消息</button>
    </div>
<script>
var app4 = new Vue({
    el:'#app-4',
    data: {
        message: 'Miracles happen every day.'
        },
    methods:{
            reverseMessage: function(){
                this.message = this.message.split('').reverse().join('')
        }
    }
})
</script>
</body>
</html>
```
示例说明：
- 同样需要注意script每个项之间需要逗号隔开
- 点击`反转消息`按钮后，文本会显示成：.yad yreve neppah selcariM
- 在`reverseMessage`方法中，更新了应用的状态，但没有触碰DOM，所有的DOM操作都由Vue来处理，编写的代码只需要关注逻辑层面

### 指令v-model
指令`v-model`能实现表单输入和应用状态之间的双向绑定，示例：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue v-model示例</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app-5">
        <p>{{ message }}</p>
        <input v-model="message">
    </div>
<script>
var app5 = new Vue({
    el:'#app-5',
    data: {
        message: 'Batman'
        }
})
</script>
</body>
</html>
```
示例说明：
- 在`input`对话框中输入啥内容，上面文本就显示啥内容

## 组件化应用构建
&#8195;&#8195;组件系统是Vue的一个重要概念，允许开发者使用小型、独立和通常可复用的组件构建大型应用。在Vue中，一个组件本质上是一个拥有预定义选项的Vue实例。Vue中注册组件很简单：
```js
//定义名为toto-item的新组件
Vue.component('todo-item',{
    template:'<li>这是个代办项</li>'
})

var app = new Vue(...)
```
创建一个todo-item组件的实例：
```html
<ol>
  <todo-item></todo-item>
</ol>
```
&#8195;&#8195;这样会为每个代办项渲染同样的文本，功能需求能从父作用域将数据传到子组件。修改一下组件的定义，使之能够接受一个prop：
```js
Vue.component('todo-item', {
  //todo-item组件现在接受一个"prop"，名为todo,类似于一个自定义attribute。
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```
使用`v-bind`指令将待办项传到循环输出的每个组件中：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue 组件构建示例</title>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
    <div id="app-6">
        <ol>
        <!-- 为每个todo-item提供todo对象,todo对象是变量，其内容可以是动态的。
        也需要为每个组件提供一个“key” -->
        <todo-item v-for="item in heroList"
            v-bind:todo="item" v-bind:key="item.id"> 
        </todo-item>
        </ol>
    </div>
<script>
Vue.component('todo-item',{
    props:['todo'],
    template: '<li>{{ todo.name }}</li>'
})
var app6 = new Vue({
    el:'#app-6',
    data: {
        heroList: [
            {id:0,name:'Captain America'},
            {id:1,name:'Iron Man'},
            {id:2,name:'Wonder Woman'}
        ]
    }
})
</script>
</body>
</html>
```
示例说明：
- 示例中设法将应用分割成了两个更小的单元，子单元通过prop接口与父单元进行了良好的解耦
- 可以进一步改进`todo-item`组件，提供更为复杂的模板和逻辑，而不会影响到父单元
- 对比之前的`v-for`示例，效果看起来一样，不同的是HTML通过prop接口获取数据
