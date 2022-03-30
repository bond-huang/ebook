# Vue-注意事项
学习过程中记录的一些注意事项。
## 插值符注意事项
&#8195;&#8195;Vue中，插值符号用两个双括号表示：`{{ value }}`，如果使用Python开发，可能会用到jinja2，在jinja2中，变量表示方法也是两个双扩号：`{{ value }}`。可以通过Vue的delimiters修改插值符号，避免冲突。也可以修改jinja2的变量表示方法，这里示例修改Vue，例如学习基础知识时候示例：   
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
修改后如下：
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
        <p>{[ message ]}</p>
    </div>
<script>
new Vue({
    el:'#app',
    data: {
        message: 'Miracles happen every day!'
    },
	delimiters:['{[', ']}']
})
</script>
</body>
</html>
```
注意事项：
- delimiters可以插在同级别任意位置，可以在el或data前，或者在其后面
- 注意如果是插在最后一项的后面，注意在此项后面加上逗号，示例中插在data后面，data结束增加了逗号
- 如果是插在最前面或者中间，注意在delimiters结尾加上逗号：`delimiters:['{[', ']}'],`
