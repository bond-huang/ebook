# Vue-条件和列表渲染
学习过程中记录的笔记，学习教程及参考链接：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue.js runoob网站教程:[https://www.runoob.com/vue2/vue-tutorial.html](https://www.runoob.com/vue2/vue-tutorial.html)

使用在线工具：
- [runoob.com菜鸟教程在线编辑器](https://www.runoob.com/try/try.php?filename=vue2-hw)

## 条件渲染
### v-if
&#8195;&#8195;指令`v-if`用于条件性地渲染一块内容，此块内容只会在指令的表达式返回`truthy`值的时候被渲染。可以搭配`v-else`使用，在2.1版本后还支持`v-else-if`。示例如下：
```html
<p v-if="loginType === 'username'">username</p>
<p v-else-if="loginType === 'telephone'">telephone</p>
<p v-else>email</p>
```
#### 在&#60;template>元素上使用v-if条件渲染分组
&#8195;&#8195;`v-if`是一个指令，必须将它添加到一个元素上。如果想切换多个元素，可以把一个&#60;template>元素当做不可见的包裹元素，并在上面使用`v-if`。最终的渲染结果将不包含&#60;template>元素。
```html
<template v-if="ok">
  <h1>Forrest Gump</h1>
  <p>Stupid is as stupid does!</p>
  <p>Miracles happen every day!</p>
</template>
<template v-else>
  <h1>Shawshank Redemption</h1>
</template>
```
### 用key管理可复用的元素
示例如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue v-if示例</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div style="padding: 50px 100px 30px;">
    <template v-if="loginType === 'username'">
        <label>Username:</label>
        <input placeholder="Enter your username" key="username-input">
    </template>
    <template v-else>
        <label>Email: </label>
        <input placeholder="Enter your email address" key="email-input">
    </template>
    <button type="button" class="btn btn-primary" @click="chInput()">Toggle login type</button>
</div>
<script>
var app = new Vue({
    el:'#app',
    data: {
        loginType: 'username',
        },
    methods:{
        chinput:function(){
            if(this.loginType == 'username'){
                this.loginType = 'email'
            }else{
                this.loginType = 'username'
            }
            }
        }
    })
</script>
</body>
</html>
```
示例说明：
- 每次点击`Toggle login type`按钮，就会切换不同内容，即&#60;template>元素里面内容
- 示例中每次切换时，输入框都将被重新渲染。如果在input里面输入了内容，切换后重新渲染内容没有了
- 如果不使用key，用户在input里面输入了内容，点击切换，内容会保留，不会重新渲染

### v-show
&#8195;&#8195;`v-show`指令是根据条件展示元素的选项。用法大致一样，不同的是带有`v-show`的元素始终会被渲染并保留在DOM中。`v-show`只是简单地切换元素的CSS property display。`v-show`不支持&#60;template>元素。格式如下
```html
<h1 v-show="ok">Hello!</h1>
```
`v-show`使用示例如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Vue v-show示例</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
</head>
<body>
<div style="padding: 50px 100px 30px;">
    <div id="app">
        <button type="button" class="btn btn-primary" @click="click()">{{ message }}</button>
        <div v-show="isShowTable">表格内容</div>
        <div v-show="isShowChart">图表内容</div>
    </div>  
</div>
<script>
var app = new Vue({
    el:'#app',
    data: {
        isShowTable: true,
        isShowChart: false,
        message: '切换图表',
        },
    methods:{
        click:function(){
            if(this.isShowTable){
                this.message = '切换表格';
            }else{
                this.message = '切换图表';
            }
            this.isShowTable = !this.isShowTable;
            this.isShowChart = !this.isShowChart;
            }
        }
    })
</script>
</body>
</html>
```
示例说明：
- 示例中data数据中的逗号必须要有（除最后一项），函数中的那个四个分号可有可无
- 之前不是用的`click`而用的`switch`，执行后通过浏览器F12看到了错误代码：`avoid using JavaScript keyword as property name: "switch"`
- 此示例参考了CSDN博客：[https://blog.csdn.net/dadada_youzi/article/details/110238197](https://blog.csdn.net/dadada_youzi/article/details/110238197)

## 列表渲染
