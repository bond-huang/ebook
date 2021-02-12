# HTML5-基础知识
HTML5学习笔记，主要学习教程：《HTML5+CSS3+JavaScript从入门到精通(标准版)》未来科技编著。
## HTML5文档基本结构
&#8195;&#8195;HTML5文档省略了&#60;html>,&#60;head>,&#60;body>等元素，使用HTML5的DOCTYPE声明文档类型，简化&#60;meta>元素的charset属性值，省略&#60;p>元素的结束标记，使用&#60;元素/>的方式来结束&#60;meta>元素，以及&#60;br>元素等语法知识要点。   
一个简单的HTML5文档基本结构如下：
```html
<!DOCTYPE html>
<mate charset="UTF-8">
<title>HTML5文档基本结构</title>
<h1>HTML 的目标</h1>
<p>HTML5的目标是为了能够创建简单的WEB程序，书写出更简洁的HTML代码。
<br/>例如，为了使WEB应用程序的开发变得更容易，提供了很多API；为了使HTML变得更简洁，开发出新的属性、新的元素等。
```
## HTML5基本语法
与HTML4相比，HTML5在语法上发生了很大的变化。
### 内容类型
扩展名仍为“.html”或“.htm”，内容类型（ContentType）仍然为“text/html”。
### 文档类型
&#8195;&#8195;DOCTYPE命令声明文档的类型，它是HTML文档必须部分，且必须位于代码第一行。在HTML5中，文档类型声明方法如下：
```html
<!DOCTYPE html>
```
使用工具时，可以在DOCTYPE声明中加入SYSTEM识别符：
```html
<!DOCTYPE HTML SYSTEM "about:legacy-compat">
```
说明及注意：
- 在HTML5中，DOCTYPE声明方式不区分大小写，引号也不区分单引号或双引号
- 使用HTML5的DOCTYPE会触发浏览器以标准模式显示页面

### 字符编码
在HTML4中，使用meta元素定义文档的字符编码：
```html
<mate http-equiv="Content-Type" content="text/html;charset=UTF-8">
```
在HTML5中，简化了charset属性写法：
```html
<mate charset="UTF-8">
```
说明及注意：
- 以上两种方法在HTML5中都有效，但是不能混用
- 从HTML5开始，对于文件的字符编码推荐使用UTF-8

### 标记省略
在HTML5中，元素的标记可以省略，分三种类型：
- 不允许写结束标记的元素：area,base,br,col,command,embed,hr,img,input,keygen,link,meta,param,source,track,wbr
- 可以省略结束标记的元素：li,dt,dd,p,rt,rp,optgroup,option,colgroup,thead,tbody,tfoot,tr,td,th
- 可以省略全部标记的元素：html,head,body,colgroup,tbody
