# Vue-基础知识
学习过程中记录的笔记，学习教程及参考链接：
- Vue.js官方中文文档：[https://cn.vuejs.org/v2/guide/syntax.html](https://cn.vuejs.org/v2/guide/syntax.html)
- Vue.js runoob网站教程:[https://www.runoob.com/vue2/vue-tutorial.html](https://www.runoob.com/vue2/vue-tutorial.html)

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
## 声明式渲染
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
