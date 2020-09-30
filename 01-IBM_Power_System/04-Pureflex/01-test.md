# HTML-基础学习笔记
学习HTML记得基础笔记。
## 基础知识
### 标签和元素
&#8195;&#8195;元素里面有：属性、内容、嵌套标签和自封闭标签。有些标签不是成对出现，例如换行标签：`<br>`。例如下面示例中`<title>`就是标签，下面整个内容叫做元素：
```html
<title>"Interstellar"</title>
```
#### 元素
分类：
- 块级元素
- 行内元素

### 基本框架
```html
<!DOCTYPE html>
<html lang="en"> 
    <head>
        <meta charset="UTF-8">
        <title>"Interstellar"</title>
    </head>
    <body>

    </body>
</html>
```
说明:
- `<!DOCTYPE html>`：文档声明标签，此为html5的
- `lang="en"`:网页使用文字的语言，属性的值用双引号或者单引号
- `<head>`:不呈现在正文部分，`<title>`是网页标题
- `<body>`:正文部分，就是网页页面呈现的内容
- `<meta charset="UTF-8">`:声明网页编码模式
