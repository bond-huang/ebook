# Bootstrap-下载及使用
学习过程中记录的笔记，学习教程及参考链接：
- [runoob.com Bootstrap教程](https://www.runoob.com/bootstrap/bootstrap-environment-setup.html)
- [bootcss.com Bootstrap教程](https://v3.bootcss.com/getting-started/)
- [bootcss.com Bootstrap教程](https://v3.bootcss.com/getting-started/)
- [Bootstrap官方教程](https://getbootstrap.com/docs/5.0/getting-started/download/)
- [bootcss.com 图标使用](https://icons.bootcss.com/)

使用在线工具：
- [HTML/CSS/JS 在线工具](https://c.runoob.com/front-end/61)
- [Bootstrap可视化布局系统](https://www.bootcss.com/p/layoutit/)

## 导航
### 示例一
根据教程写了一个简单网页头部导航：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>WEB导航器</title>
    <link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body style="background: #f0f0f0">
    <nav class="navbar navbar-default navbar-static-top" role="navigation">
    <div class="container-fluid"> 
       <div class="navbar-header">
            <a class="navbar-brand">
            <i class="bi bi-cursor-fill" style="font-size: 3rem; color: blue;"></i>
            </a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav">                    
                <li class="active">
                    <a href="#">网站导航器</a>
                </li>
            </ul>
            <a class="btn pull-right js-toolbar-action" href="https://github.com/bond-huang/ebook">
                <i class="bi bi-github"></i>
            </a>
        </div>
    </div>
    </nav>
</body>
</html>
```
说明：
- 前面是一个导航图标：bi-cursor-fill
- 然后是网页名称，用列表形式做的
- 最后是github图标，在导航栏最右边，点击转向指定网站

### 示例二
同样是横向的导航，增加了一个搜索框：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>WEB导航器</title>
    <link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
    <nav class="navbar navbar-default navbar-static-top" role="navigation"> 
        <ul class="nav nav-pills">
            <li role="presentation" class="active"><a href="#">
            <i class="bi bi-house-fill"></i> Home</a></li>
            <li role="presentation"><a href="#">Profile</a></li>
            <li role="presentation"><a href="#">Messages</a></li>
            <li class="dropdown">
            <a class="dropdown-toggle" data-toggle="dropdown" href="#">
            More <span class="caret"></span>
            </a>
                <ul class="dropdown-menu">
                    <li><a href="#">About</a></li>
                    <li><a href="#">License</a></li>
                    <li><a href="#">Other</a></li>
                    <li class="divider"></li>
                    <li><a href="#">Contact us</a></li>
                </ul>
            </li>
        <div class="col search row-search-mobile">
            <form class="navbar-form navbar-right">
        <div class="form-group">
            <input type="text" class="form-control" placeholder="Key Words">
        </div>
            <button type="submit" class="btn btn-default">Search</button>
            </form>
        </div>
        </ul>
    </nav>
</body>
</html>
```
示例说明：
- 示例中采用横向显示的导航菜单，最后一个元素带下拉菜单
- 增加了一个搜索框在最右边

### 示例三
垂直导航示例：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <title>WEB导航器</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body style="background: #f0f0f0">
    <div class="container-fluid">
        <div class="row-fluid">
            <div class="col-md-2">
            <ul class="nav nav-list">
                <li class="nav-header">分类列表</li>
                <li class="active"><a href="#">小型机</a></li>
                <li><a href="#">存储设备</a></li>
                <li><a href="#">交换机</a></li>
                <li class="nav-header">功能列表</li>
                <li><a href="#">添加</a></li>
                <li><a href="#">导入</a></li>
                <li class="divider"></li>
                <li><a href="#">帮助</a></li>
            </ul>
            </div>
            <div class="col-md-10">
            <h2>Forrest Gump</h2>
            <p>
            Life was like a box of chocolates, you never know what you're gonna get.
            </p>
            <p><a class="btn" href="#">查看更多 »</a></p>
            </div>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/jquery@1.12.4/dist/jquery.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/js/bootstrap.min.js"></script>
</body>
</html>
```
示例说明：
- 导航菜单采用垂直显示方式，并且使用`.col-md-*`栅格类分配了2格位置
- 右边分配了10格位置，用于显示正文

### 导航条
导航条是在网站中作为导航页头的响应式基础组件，导航条示例如下：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>网页导航</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.4.0/font/bootstrap-icons.css">
    <script src="https://cdn.bootcss.com/jquery/2.2.4/jquery.min.js"></script>
    <script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body style="background: #f0f0f0">
    <nav class="navbar navbar-default">
        <div class="container-fluid">
        <div class="navbar-header">
        <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
        </button>
        <a class="navbar-brand" href="#"><i class="bi bi-cursor-fill" style="font-size: 2rem; color: blue;"></i>&nbsp;&nbsp;<strong>My Navigator </strong></a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
        <ul class="nav navbar-nav">
            <li class="active"><a href="#"><i class="bi bi-house-fill"></i>&nbsp;&nbsp;Home<span class="sr-only">(current)</span></a></li>
            <li><a href="#">Profile</a></li>
            <li><a href="#">Message</a></li>
            <li class="dropdown">
                <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Action<span class="caret"></span></a>
                <ul class="dropdown-menu">
                <li><a href="#"><i class="bi bi-plus-circle"></i>&nbsp;&nbsp;Add</a></li>
                <li><a href="#"><i class="bi bi-arrow-down-up"></i>&nbsp;&nbsp;Import</a></li>
                <li><a href="#"><i class="bi bi-trash"></i>&nbsp;&nbsp;Delete</a></li>
                <li role="separator" class="divider"></li>
                <li><a href="#"><i class="bi bi-arrow-repeat"></i>&nbsp;&nbsp;Refresh</a></li>
                <li role="separator" class="divider"></li>
                <li><a href="#"><i class="bi bi-download"></i>&nbsp;&nbsp;Download</a></li>
                </ul>
            </li>
        </ul>
        <ul class="nav navbar-nav navbar-right">
            <li class="dropdown">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">About<span class="caret"></span></a>
            <ul class="dropdown-menu">
                <li><a href="#"><i class="bi bi-share-fill"></i>&nbsp;&nbsp;Share</a></li>
                <li><a href="#"><i class="bi bi-github"></i>&nbsp;&nbsp;GitHub</a></li>
                <li><a href="#"><i class="bi bi-envelope-fill"></i>&nbsp;&nbsp;Email</a></li>
                <li role="separator" class="divider"></li>
                <li><a href="#">&nbsp;&nbsp;View License</a></li>
            </ul>
            </li>
        </ul>
        <form class="navbar-form navbar-right">
        <div class="form-group">
            <input type="text" class="form-control" placeholder="Key Words">
        </div>
        <button type="submit" class="btn btn-default"><i class="bi bi-search"></i>&nbsp;&nbsp;Search</button>
        </form>
        </div>
        </div>
    </nav>
</body>
</html>
```
