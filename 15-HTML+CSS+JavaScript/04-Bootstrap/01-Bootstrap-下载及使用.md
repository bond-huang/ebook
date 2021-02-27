# Bootstrap-下载及使用
学习过程中记录的笔记，学习教程及参考链接：
- [runoob.com Bootstrap教程](https://www.runoob.com/bootstrap/bootstrap-environment-setup.html)
- [bootcss.com Bootstrap教程](https://v3.bootcss.com/getting-started/)
- [bootcss.com Bootstrap教程](https://v3.bootcss.com/getting-started/)
- [Bootstrap官方教程](https://getbootstrap.com/docs/5.0/getting-started/download/)
- [bootcss.com 图标使用](https://icons.bootcss.com/)

## 简介
&#8195;&#8195;Bootstrap是一个用于快速开发Web应用程序和网站的前端框架。Bootstrap是基于HTML、CSS、JAVASCRIPT的。Bootstrap包的内容：
- 基本结构：提供了一个带有网格系统、链接样式、背景的基本结构
- CSS：全局的CSS设置、定义基本的HTML元素样式、可扩展的class，以及一个先进的网格系统
- 组件：包含了多个可重用的组件，用于创建图像、下拉菜单、导航、警告框、弹出框等等
- JavaScript插件：包含了多个自定义的jQuery插件
- 定制：可以定制Bootstrap的组件、LESS变量和 Query插件来得到自己的版本

## 下载
相关下载链接：
- Bootstrap官方下载地址：[Bootstrap官方下载链接](https://getbootstrap.com/docs/5.0/getting-started/download/)
- jQuery官方下载链接：[https://jquery.com/download/](https://jquery.com/download/)

Bootstrap下载说明：
- 下载已编译的版本：bootstrap-5.0.0-beta2-dist.zip
- 解压后即可使用，一共两个文件夹：css和js
- 文件中`bootstrap.*`是已编译的CSS和JS
- 文件中`bootstrap.min.*`是已编译及压缩的CSS和JS

jQuery下载说明同上，还有其它下载安装方法，可以参考官方文档说明。
## 使用方法
### 通过下载文件使用
根据上面方法下载文件后，直接拷贝到文件夹中即可使用，示例：
```html
<link href="./bootstrap/css/bootstrap.css" type="text/css" rel="stylesheet">
<!-- JavaScript放置在后面可以使页面加载速度更快 -->
<script type="text/javascript" src="./bootstrap/jquery/jquery-3.5.1.js"></script>
<script type="text/javascript" src="./bootstrap/js/bootstrap.js"></script>
```
### 通过CDN服务使用
其实就是在线引用，不需要下载，官方文档示例如下：
```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-BmbxuPwQa2lc/FVzBcNJ7UAyJxM6wuqIj61tLrc4wSX0szH/Ev+nYRRuWlolflfl" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.0-beta2/dist/js/bootstrap.bundle.min.js" integrity="sha384-b5kHyXgcpbZJO/tY9Ul7kGkf1S0CWuKcCD38l8YkeH8z8QjE0GmW1gYU5S9FOnJ0" crossorigin="anonymous"></script>
```
国内推荐使用Staticfile CDN上的库，使用示例：
```html
<link href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
``` 
国内推荐CDN服务：[http://staticfile.org/](http://staticfile.org/)    
国外推荐CDN服务：[https://cdnjs.com/](https://cdnjs.com/)    
官方文档使用CDN：[https://www.jsdelivr.com/](https://www.jsdelivr.com/)

## bootstrap图标
Bootstrap图标库：[https://icons.bootcss.com/](https://icons.bootcss.com/)        
下载地址：[https://github.com/twbs/icons/releases/tag/v1.4.0](https://github.com/twbs/icons/releases/tag/v1.4.0)

使用同样可以下载后在HTML中加载，也可以通过公共CDN加载，示例:
```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css");
```
### 使用方法示例
内嵌方式：
```html
<svg class="bi bi-chevron-right" width="32" height="32" viewBox="0 0 20 20" fill="currentColor" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M6.646 3.646a.5.5 0 01.708 0l6 6a.5.5 0 010 .708l-6 6a.5.5 0 01-.708-.708L12.293 10 6.646 4.354a.5.5 0 010-.708z"/></svg>
```
利用SVG sprite和&#60;use>元素即可插入任何图标:
```html
<svg class="bi" width="32" height="32" fill="currentColor">
  <use xlink:href="bootstrap-icons.svg#heart-fill"/>
</svg>
<svg class="bi" width="32" height="32" fill="currentColor">
  <use xlink:href="bootstrap-icons.svg#toggles"/>
</svg>
<svg class="bi" width="32" height="32" fill="currentColor">
  <use xlink:href="bootstrap-icons.svg#shop"/>
</svg>
```
作为外部图片引用：
```html
<img src="./bootstrap-icons/bootstrap.svg" alt="Bootstrap" width="32" height="32">
```
使用HTML标签添加对应的class名称：
```html
<i class="bi-tree-fill"></i>
<i class="bi-tree-fill" style="font-size: 2rem; color: green;"></i>
```
CSS方式,在CSS中使用SVG图标：
```css
.bi::before {
  display: inline-block;
  content: "";
  vertical-align: -.125em;
  background-image: url("data:image/svg+xml,<svg viewBox='0 0 16 16' fill='%23333' xmlns='http://www.w3.org/2000/svg'><path fill-rule='evenodd' d='M8 9.5a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3z' clip-rule='evenodd'/></svg>");
  background-repeat: no-repeat;
  background-size: 1rem 1rem;
}
```
可访问性：如果图标不是纯装饰性的，确保提供适当的替代性文本:
```html
<img src="/assets/img/bootstrap.svg" alt="Bootstrap" ...>
```
当图标获取不到时候，就会显示文本Bootstrap。
## 使用示例及说明
### 使用示例一
使用示例如下：
```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
    <title>Bootstrap使用测试</title>
	<!-- 包含头部信息用于适应不同设备 -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="./bootstrap/css/bootstrap.css">
    <!-- bootstrap图标 -->
	<link rel="stylesheet" href="./bootstrap-icons/bootstrap-icons.css">
</head>
<body>
	<div class="container">
	<h2>示例表格</h2>
	<p>The system disk performance is follow:</p>
	<!-- 创建响应式表格 (将在小于768px的小型设备下水平滚动) -->
	<div class="table-responsive">          
       <table class="table table-striped table-bordered">
	   <thead>
		<tr>
			<th rowspan="2">Disk</th>
			<th colspan="2">kbps</th>
			<th colspan="2">tps</th>
		</tr>
		<tr>
			<th>max</th>
			<th>avg</th>
			<th>max</th>
			<th>avg</th>
		</tr>
		</thead>
		<tbody>
		<tr>
			<td>hdisk2</td>
			<td>1.4</td>
			<td>1.1</td>
			<td>5</td>
			<td>1.7</td>
		</tr>
		</tbody>
		</table>
	</div>
		<h2>图标示例</h2> 
		<!-- 从bootstrap-icons.css中获取图标 -->
		<p>这是一棵树：<i class="bi bi-tree-fill"></i></p>
		<p>这是一棵大绿树：<i class="bi bi-tree-fill" style="font-size: 2rem; color: green;"></i></p>
		<!-- 作为外部图片文件引用 -->
		<p>闹钟图标：<img src="./bootstrap-icons/alarm.svg" alt="Bootstrap" width="25" height="25"></p>
	</div>
	<!-- JavaScript放置在后面可以使页面加载速度更快 -->
    <script type="text/javascript" src="./bootstrap/jquery/jquery-3.5.1.js"></script>
	<script type="text/javascript" src="./bootstrap/js/bootstrap.js"></script>
</body>
</html>
```
示例说明：
- 示例中使用的是加载本地的CSS文件和JS文件，图标也是单独下载的图标库
- 示例中使用了响应式Web设计，是一个让用户通过各种尺寸的设备浏览网站获得良好的视觉效果的方法
- 示例中使用了thead和tbody元素，默认不会影响表格的布局，可以使用CSS来为这些元素定义样式，从而改变表格的外观
- 务必在bootstrap.js之前引入jQuery文件，Bootstrap的所有JavaScript插件都依赖jQuery

关于响应式设计的说明：[Bootstrap响应式设计](https://www.runoob.com/bootstrap/bootstrap-v2-responsive-design.html)

### 使用示例二
示例代码如下：
```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
    <title>Bootstrap使用测试</title>
	<!-- 包含头部信息用于适应不同设备 -->
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://apps.bdimg.com/libs/bootstrap/3.2.0/css/bootstrap.min.css">
</head>
<body>
	<div class="container">
	<h2>图像示例</h2>
	  <p>图片以椭圆型展示：</p>
	  <!-- 创建响应式图片(将扩展到父元素) -->
	  <img src="erhai.jpg" class="img-responsive img-circle" alt="洱海" width="304" height="236"> 
    <h2>图标示例</h2>  
      <p>云图标: <span class="glyphicon glyphicon-cloud"></span></p>      
      <p>信件图标: <span class="glyphicon glyphicon-envelope"></span></p>            
      <p>搜索图标: <span class="glyphicon glyphicon-search"></span></p>
      <p>打印图标: <span class="glyphicon glyphicon-print"></span></p>      
      <p>下载图标：<span class="glyphicon glyphicon-download"></span></p>  
	</div>
    <script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
    <script src="https://apps.bdimg.com/libs/bootstrap/3.2.0/js/bootstrap.min.js"></script>
</body>
</html>
```
示例说明：
- 示例中采用线上加载CSS和JS文件的，版本有点老
- 示例中加载的CSS文件中有一些图标，可以直接使用
